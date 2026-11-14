---
title: "Day 49. 종합 — 홈랩 K8s에 사내 문서 RAG 챗봇 운영하기"
date: 2026-11-15T03:14:35+09:00
weight: 49
tags: ["ai-infra", "llm", "rag", "kubernetes"]
summary: "Day 36~48에서 만든 부품(GPU Operator, vLLM, 임베딩, Qdrant, rag-api, LiteLLM, Open WebUI, 모니터링, 보안)을 하나의 서비스로 조립한다. 매니페스트·Helm values를 디렉터리 하나에 정리하고, 배포 순서대로 검증하며 올린다."
image: "./images/day49-capstone-architecture.svg"
draft: false
---
> **Chapter 6 · AI 인프라** | 난이도 ★★★★★ | 예상 소요 5 ~ 7시간 (하루 또는 주말)

## 🎯 오늘의 목표

- Day 36~48에서 만든 부품(GPU Operator, vLLM, 임베딩, Qdrant, rag-api, LiteLLM, Open WebUI, 모니터링, 보안)을 **하나의 서비스**로 조립한다
- 매니페스트·Helm values를 **디렉터리 하나**에 정리하고, **배포 순서**대로 검증하며 올린다
- 문서가 바뀌면 자동으로 다시 인덱싱하는 **CronJob**을 만든다
- **Cloudflare Tunnel**(H03)로 포트를 열지 않고 공개하고, 운영 체크리스트와 **비용**을 정리한다
- Chapter 0~6을 돌아보고 다음 단계(Chapter 7 GitOps)를 준비한다

---

## 1. 무엇을 만드는가

> **"사내(=이 레포) 문서를 근거로 답하는 챗봇을, 홈랩 서버 한 대의 Kubernetes에서, 외부에서도 안전하게 쓸 수 있게 운영한다."**

| 요구사항 | 구현 | 근거 Day |
|---|---|---|
| 웹 채팅 UI, 로그인 | Open WebUI | 38 |
| 문서 근거 답변 + 출처 | rag-api (FastAPI) + Qdrant | 41, 42, 43 |
| LLM 추론 | vLLM (GPU) | 39, 45 |
| 임베딩 | Text Embeddings Inference (CPU) | 41 |
| 모델 통합·키·예산 | LiteLLM 게이트웨이 | 40, 48 |
| GPU 스케줄링 | NVIDIA GPU Operator | 37, 44 |
| 관측·알림 | kube-prometheus-stack + DCGM + Langfuse | 17, 47 |
| 보안 | PII 마스킹, 출력 검증, NetworkPolicy, 가상 키 | 19, 48 |
| 문서 변경 반영 | 재인덱싱 CronJob | 오늘 |
| 외부 공개 | Cloudflare Tunnel + Cloudflare Access | H03 |

![사내 문서 RAG 챗봇 전체 아키텍처](./images/day49-capstone-architecture.svg)

**요청 한 번의 흐름**: 사용자 → Cloudflare(Access 인증) → cloudflared → Open WebUI → rag-api(`/v1/chat/completions`) → ① LiteLLM → TEI로 질문 임베딩 → ② Qdrant 검색 → ③ LiteLLM → vLLM으로 답변 생성 → 출력 검증 → Open WebUI에 출처와 함께 표시. 그동안 Prometheus는 vLLM·GPU 지표를 긁고, rag-api는 Langfuse로 트레이스를 보냅니다.

---

## 2. 하드웨어와 리소스 계획

| 항목 | 최소 | 권장 | 비고 |
|---|---|---|---|
| 서버 | H01의 홈랩 PC 1대 (Ubuntu 24.04) | 같은 구성 | 단일 노드 K8s (control plane taint 제거) |
| GPU | VRAM 8GB (RTX 3060/4060급) | VRAM 16GB 이상 | 8GB면 1.5B 모델, 16GB면 4B~7B급 |
| RAM | 32GB | 64GB | TEI(CPU) · Qdrant · Prometheus · Langfuse가 메모리를 씀 |
| 디스크 | SSD 200GB | NVMe 500GB | 모델 캐시, Prometheus, 컨테이너 이미지 |

| 워크로드 | 네임스페이스 | 자원 (요청) | 설치 방식 |
|---|---|---|---|
| vLLM `vllm-qwen` (`Qwen/Qwen2.5-1.5B-Instruct`, Day 45) | `llm` | GPU 1 | 매니페스트 (Day 45 그대로) |
| TEI `BAAI/bge-m3` (Day 43과 같은 모델) | `llm` | CPU 2, 메모리 3Gi | 매니페스트 |
| LiteLLM + PostgreSQL | `gateway` | CPU 0.5, 메모리 1Gi | 매니페스트 |
| Qdrant | `rag` | CPU 0.5, 메모리 1Gi, PVC 10Gi | Helm (`qdrant/qdrant`) |
| rag-api ×2, 재인덱싱 CronJob | `rag` | CPU 0.5, 메모리 512Mi | 매니페스트 |
| Open WebUI | `webui` | CPU 0.5, 메모리 1Gi, PVC 5Gi | Helm (`open-webui/open-webui`) |
| cloudflared ×2 | `edge` | CPU 0.1, 메모리 128Mi | 매니페스트 |
| GPU Operator, kube-prometheus-stack | `gpu-operator`, `monitoring` | - | Helm |

> 💡 임베딩을 **CPU**에서 돌리는 이유: GPU가 1장뿐이면 VRAM은 LLM의 KV 캐시에 최대한 몰아주는 게 이득입니다. bge-m3(약 0.6B) 임베딩 모델은 CPU로도 질문 1개(수십 토큰)를 수십 ms에 처리합니다. 대량 재인덱싱 때만 느린데, 그건 새벽 CronJob이 감당합니다.

> 💡 **GPU가 없다면** — 두 가지 길이 있습니다. ① 클라우드 GPU VM 한 대를 빌려 오늘 실습을 그대로(주말 하루 비용이면 충분), ② vLLM 대신 **Ollama CPU 모드**(Day 38)를 `llm` 네임스페이스에 올리고 LiteLLM의 `api_base`만 Ollama로 바꿉니다. 게이트웨이 덕분에 rag-api와 Open WebUI는 **한 줄도 바뀌지 않습니다.** 이것이 Day 40에서 게이트웨이를 둔 이유입니다.

---

## 3. 저장소 디렉터리 구조

지금까지 Day마다 흩어져 있던 YAML을 **배포 순서 번호**를 붙인 디렉터리 하나로 모읍니다. Chapter 7에서는 이 디렉터리를 그대로 Git 저장소로 만들어 Argo CD가 동기화하게 됩니다.

```
homelab-rag/
├── README.md                       # 이 문서의 체크리스트
├── 00-cluster/
│   └── kubeadm-config.yaml         # Day 20 (단일 노드) + Calico
├── 10-platform/
│   ├── gpu-operator-values.yaml    # Day 44
│   └── kube-prometheus-stack-values.yaml   # Day 17
├── 20-llm/
│   ├── namespace.yaml
│   ├── vllm.yaml                   # Day 45 그대로: hf-cache PVC + vllm-qwen Deployment/Service/ServiceMonitor
│   ├── embed.yaml                  # TEI Deployment + Service
│   └── netpol.yaml                 # Day 48
├── 30-gateway/
│   ├── litellm-config.yaml         # 모델 목록, 가드레일, 단가
│   ├── litellm.yaml                # PostgreSQL + LiteLLM Deployment/Service
│   └── secrets.example.yaml        # 실제 값은 Git에 넣지 않음
├── 40-rag/
│   ├── qdrant-values.yaml
│   ├── rag-api.yaml                # Deployment ×2 + Service + Secret 참조
│   ├── reindex-cronjob.yaml        # 오늘
│   └── netpol.yaml                 # Day 48 + 오늘 추가분
├── 50-webui/
│   └── open-webui-values.yaml
├── 60-edge/
│   └── cloudflared.yaml            # H03 → K8s
├── 70-observability/
│   └── llm-alerts.yaml             # Day 47 PrometheusRule
└── rag/                            # rag-api 소스와 Dockerfile (Day 42~43, 이미지 infra-rag)
    ├── Dockerfile  requirements.txt
    ├── common.py  ingest.py  app.py  reindex.py
    ├── pii.py  scan.py  output_guard.py  tracing.py
    └── eval/  golden.jsonl  eval_retrieval.py  eval_ragas.py
```

> ⚠️ `secrets.example.yaml`에는 **키 이름만** 두고, 실제 값은 `kubectl create secret`으로 넣습니다(Day 13). HF 토큰, LiteLLM 마스터 키, Qdrant API 키, Langfuse 키, Cloudflare 터널 토큰이 Git에 올라가면 사고입니다. (Sealed Secrets·External Secrets 같은 도구로 Secret까지 Git과 함께 관리하는 방법도 있습니다.)

---

## 4. 배포 순서 체크리스트

![배포 순서와 단계별 검증](./images/day49-deploy-order.svg)

원칙은 **"아래층부터, 한 층 올릴 때마다 검증"** 입니다. 문제가 생겼을 때 방금 올린 층만 보면 되니까요.

### ☐ 0단계 — 클러스터 (H01 + Day 20)

```bash
# 단일 노드: control plane에도 Pod가 뜨도록 taint 제거
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl get nodes        # STATUS Ready
kubectl get pods -n kube-system | grep -E 'calico|coredns'   # 모두 Running
```

NetworkPolicy를 쓰려면 Calico처럼 이를 지원하는 CNI가 필요하고, PVC를 쓰려면 기본 StorageClass가 있어야 합니다(Day 14, 예: local-path-provisioner).

### ☐ 1단계 — GPU Operator (Day 44)

```bash
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update
helm install gpu-operator nvidia/gpu-operator -n gpu-operator --create-namespace \
  --version v26.7.2 \
  --set dcgmExporter.serviceMonitor.additionalLabels.release=monitoring

kubectl get node -o jsonpath='{.items[0].status.allocatable.nvidia\.com/gpu}{"\n"}'
# 1
```

> 💡 H01 서버에 NVIDIA 드라이버를 Day 37에서 이미 설치했다면 `--set driver.enabled=false`를 추가합니다.

### ☐ 2단계 — 모니터링 (Day 17)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace -f 10-platform/kube-prometheus-stack-values.yaml
kubectl -n monitoring get pods     # prometheus-…, grafana-…, alertmanager-… Running
```

### ☐ 3단계 — LLM과 임베딩 (`20-llm/`)

```bash
kubectl create namespace llm
kubectl -n llm create secret generic hf-token --from-literal=HF_TOKEN=hf_xxx   # 게이티드 모델일 때만 필요
kubectl apply -f 20-llm/
kubectl -n llm rollout status deploy/vllm-qwen --timeout=15m    # 첫 실행은 모델 다운로드로 오래 걸림
kubectl -n llm rollout status deploy/embed --timeout=10m
```

`20-llm/vllm.yaml`은 **Day 45의 매니페스트를 그대로** 씁니다: `hf-cache` PVC, 모델 사전 다운로드 Job, `vllm-qwen` Deployment(`vllm/vllm-openai:v0.33.0`, `--served-model-name=qwen2.5-1.5b`, GPU 1장이라 `strategy: Recreate`), 포트 이름 `http`의 Service, `release: monitoring` 라벨의 ServiceMonitor. Day 46의 KEDA ScaledObject는 GPU가 1장인 오늘 환경에서는 빼 둡니다(최대 레플리카 1).

`20-llm/embed.yaml` (TEI CPU 이미지, 포트를 8000으로 통일):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: embed, namespace: llm, labels: { app: embed } }
spec:
  replicas: 1
  selector: { matchLabels: { app: embed } }
  template:
    metadata: { labels: { app: embed } }
    spec:
      containers:
        - name: tei
          image: ghcr.io/huggingface/text-embeddings-inference:cpu-1.9
          args: ["--model-id", "BAAI/bge-m3", "--port", "8000"]
          ports: [{ name: http, containerPort: 8000 }]
          resources: { requests: { cpu: "2", memory: 3Gi } }
          volumeMounts: [{ name: data, mountPath: /data }]
      volumes: [{ name: data, emptyDir: {} }]
---
apiVersion: v1
kind: Service
metadata: { name: embed, namespace: llm, labels: { app: embed } }
spec:
  selector: { app: embed }
  ports: [{ name: http, port: 8000, targetPort: 8000 }]
```

```bash
# 검증: 두 서버 모두 OpenAI 호환 응답
kubectl -n llm run t --rm -it --image=curlimages/curl:8.23.0 --restart=Never -- \
  curl -s http://vllm-qwen:8000/v1/models
kubectl -n llm run t --rm -it --image=curlimages/curl:8.23.0 --restart=Never -- \
  curl -s http://embed:8000/v1/embeddings -H 'Content-Type: application/json' \
  -d '{"model":"bge-m3","input":"hello"}' | head -c 120
```

### ☐ 4단계 — 게이트웨이 (`30-gateway/`)

`30-gateway/litellm-config.yaml`:

```yaml
model_list:
  - model_name: qwen2.5-1.5b
    litellm_params:
      model: hosted_vllm/qwen2.5-1.5b          # vLLM의 --served-model-name (Day 40·45)
      api_base: http://vllm-qwen.llm.svc.cluster.local:8000/v1
      input_cost_per_token: 0.0000004     # 내부 원가 ≈ $0.4/1M (8장 계산), 예산 기능용
      output_cost_per_token: 0.0000016    # ≈ $1.6/1M
  - model_name: embed
    litellm_params:
      model: openai/bge-m3                    # TEI의 OpenAI 호환 /v1/embeddings
      api_base: http://embed.llm.svc.cluster.local:8000/v1
      api_key: "none"

litellm_settings:
  success_callback: ["langfuse"]          # Day 47

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  database_url: os.environ/DATABASE_URL
```

```bash
kubectl create namespace gateway
kubectl -n gateway create configmap litellm-config --from-file=config.yaml=30-gateway/litellm-config.yaml
kubectl -n gateway create secret generic litellm-env \
  --from-literal=LITELLM_MASTER_KEY=sk-master-$(openssl rand -hex 16) \
  --from-literal=LITELLM_SALT_KEY=$(openssl rand -hex 32) \
  --from-literal=DATABASE_URL=postgresql://litellm:CHANGEME@litellm-db:5432/litellm \
  --from-literal=LANGFUSE_PUBLIC_KEY=pk-lf-xxx --from-literal=LANGFUSE_SECRET_KEY=sk-lf-xxx \
  --from-literal=LANGFUSE_HOST=http://192.168.0.10:3000
kubectl apply -f 30-gateway/litellm.yaml     # 이미지: ghcr.io/berriai/litellm:v1.109.1, args: --config /app/config.yaml

# 앱별 가상 키 발급 (Day 48 6장): rag-api용, Open WebUI용
kubectl -n gateway port-forward svc/litellm 4000:4000 &
curl -s localhost:4000/key/generate -H "Authorization: Bearer $MASTER" -H 'Content-Type: application/json' \
  -d '{"key_alias":"rag-api","models":["qwen2.5-1.5b","embed"],"max_budget":5,"budget_duration":"30d","rpm_limit":120}'
```

### ☐ 5단계 — 벡터 DB와 rag-api (`40-rag/`)

```bash
helm repo add qdrant https://qdrant.github.io/qdrant-helm && helm repo update
kubectl create namespace rag
helm install qdrant qdrant/qdrant -n rag --version 1.20.1 -f 40-rag/qdrant-values.yaml
#   qdrant-values.yaml:  apiKey: "<랜덤 문자열>"  /  persistence: { size: 10Gi }

# Day 42 common.py의 환경 변수 이름 그대로 — LLM과 임베딩 모두 게이트웨이 경유
kubectl -n rag create secret generic rag-api-env \
  --from-literal=LLM_BASE_URL=http://litellm.gateway.svc.cluster.local:4000/v1 \
  --from-literal=LLM_API_KEY=sk-<rag-api 가상 키> \
  --from-literal=LLM_MODEL=qwen2.5-1.5b \
  --from-literal=EMBED_BASE_URL=http://litellm.gateway.svc.cluster.local:4000/v1 \
  --from-literal=EMBED_API_KEY=sk-<rag-api 가상 키> \
  --from-literal=EMBED_MODEL=embed \
  --from-literal=EMBED_DIM=1024 \
  --from-literal=COLLECTION=infra_docs \
  --from-literal=QDRANT_URL=http://qdrant.rag.svc.cluster.local:6333 \
  --from-literal=QDRANT_API_KEY=<qdrant 키>
kubectl apply -f 40-rag/rag-api.yaml          # 이미지: Day 43에서 빌드한 infra-rag:0.1을 레지스트리에 push한 것
kubectl -n rag apply -f 40-rag/reindex-cronjob.yaml
kubectl -n rag create job initial-index --from=cronjob/reindex   # 5장의 CronJob으로 최초 인덱싱
kubectl -n rag logs -f job/initial-index -c indexer
# [UPD] 00-homelab/homelab01-ubuntu-server.md (23 chunks)
# ...
# done: 60 changed, 0 deleted, 0 unchanged
```

> ⚠️ Day 42의 `common.py`는 `QdrantClient(url=QDRANT_URL)`로 키 없이 접속합니다. Qdrant에 `apiKey`를 켰으므로 `QdrantClient(url=QDRANT_URL, api_key=os.getenv("QDRANT_API_KEY"))`로 한 줄 고쳐 이미지를 다시 빌드하세요.

Open WebUI가 rag-api를 **"모델 하나"** 로 인식하도록, rag-api에 OpenAI 호환 엔드포인트 두 개를 얇게 추가합니다(Day 42의 `ask()`를 재사용 — Day 43에서 예고한 그 방법입니다).

```python
# app.py 에 추가 — Open WebUI 연동용 OpenAI 호환 래퍼
import json
import time
import uuid

from fastapi.responses import StreamingResponse


class ChatMessage(BaseModel):
    role: str
    content: str


class ChatRequest(BaseModel):
    model: str
    messages: list[ChatMessage]
    stream: bool = False


@app.get("/v1/models")
def list_models():
    return {"object": "list", "data": [{"id": "docs-rag", "object": "model", "created": 0, "owned_by": "homelab"}]}


@app.post("/v1/chat/completions")
def chat_completions(req: ChatRequest):
    question = next(m.content for m in reversed(req.messages) if m.role == "user")
    result = ask(AskRequest(question=question, stream=False))   # 검색 → 생성 → 출력 검증 (Day 42·48)
    content = result["answer"] + "\n\n**출처**\n" + "\n".join(
        f"- `{s['source']}` — {s['section']}" for s in result["sources"])
    cid, created = f"chatcmpl-{uuid.uuid4().hex}", int(time.time())

    if not req.stream:
        return {"id": cid, "object": "chat.completion", "created": created, "model": "docs-rag",
                "choices": [{"index": 0, "message": {"role": "assistant", "content": content},
                             "finish_reason": "stop"}]}

    def sse():
        base = {"id": cid, "object": "chat.completion.chunk", "created": created, "model": "docs-rag"}
        yield "data: " + json.dumps({**base, "choices": [{"index": 0, "delta": {"role": "assistant", "content": content}, "finish_reason": None}]}, ensure_ascii=False) + "\n\n"
        yield "data: " + json.dumps({**base, "choices": [{"index": 0, "delta": {}, "finish_reason": "stop"}]}) + "\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(sse(), media_type="text/event-stream")
```

> 💡 답을 다 만든 뒤 한 번에 보내는 "가짜 스트리밍"입니다. 진짜 토큰 스트리밍은 LiteLLM 호출에 `stream=True`를 주고 받은 청크를 그대로 흘려보내면 되지만, 그러면 Day 48 5장의 **출력 검증을 스트리밍 전에 할 수 없다**는 트레이드오프가 생깁니다. 사내 문서 챗봇에서는 검증을 택했습니다.

### ☐ 6단계 — Open WebUI (`50-webui/`)

```yaml
# 50-webui/open-webui-values.yaml
ollama:
  enabled: false
pipelines:
  enabled: false
openaiBaseApiUrls:
  - "http://rag-api.rag.svc.cluster.local:8000/v1"           # docs-rag (문서 챗봇)
  - "http://litellm.gateway.svc.cluster.local:4000/v1"       # 일반 대화 · 작업 모델
openaiApiKeysExistingSecret: "webui-openai-keys"             # "unused;sk-<webui 가상 키>"
openaiApiKeysExistingSecretKey: "keys"
persistence:
  enabled: true
  size: 5Gi
```

```bash
kubectl create namespace webui
kubectl -n webui create secret generic webui-openai-keys --from-literal=keys='unused;sk-<webui 가상 키>'
helm repo add open-webui https://helm.openwebui.com/ && helm repo update
helm install open-webui open-webui/open-webui -n webui --version 18.3.0 -f 50-webui/open-webui-values.yaml
kubectl -n webui port-forward svc/open-webui 8080:80
# http://localhost:8080 → 첫 가입자가 관리자. 모델 목록에 docs-rag, qwen2.5-1.5b 가 보이면 성공
```

> ⚠️ Open WebUI는 대화 제목·후속 질문 생성 같은 **작업(task) 요청**을 선택된 모델로 보냅니다. docs-rag가 이 요청까지 RAG로 처리하면 엉뚱한 제목이 생기고 GPU를 낭비하므로, 관리자 설정에서 **작업 모델(Task Model)** 을 `qwen2.5-1.5b`로 지정하세요. 신규 가입자는 기본 역할이 `pending`이라 관리자가 승인해야 쓸 수 있습니다 — 그대로 두는 것을 권장합니다.

### ☐ 7단계 — 관측과 보안 연결 (Day 47·48)

```bash
kubectl apply -f 20-llm/servicemonitor.yaml -f 70-observability/llm-alerts.yaml
kubectl apply -f 40-rag/netpol.yaml       # llm·gateway·rag 네임스페이스 default deny + 화이트리스트
```

NetworkPolicy는 Day 48의 두 정책에 더해, `gateway` 네임스페이스로 들어오는 트래픽을 `rag`·`webui`에서만, `rag`의 rag-api로 들어오는 트래픽을 `webui`에서만 허용하도록 같은 방식으로 추가합니다. 적용 후 **반드시 Open WebUI에서 질문 하나를 끝까지** 해 보세요. 정책 하나만 빠져도 어딘가에서 타임아웃이 납니다.

### ☐ 8단계 — 외부 공개 (H03 → K8s)

6장에서 다룹니다.

---

## 5. 문서가 바뀌면 — 재인덱싱 CronJob

문서는 계속 바뀝니다. 바뀐 문서를 반영하지 않으면 챗봇은 **예전 내용을 자신 있게** 말합니다(LLM07 Misinformation). 6시간마다 레포를 받아 **바뀐 파일만** 다시 인덱싱합니다.

### 5-1. 증분 인덱서

Day 42의 `ingest.py`는 매번 컬렉션을 지우고 **전체**를 다시 넣습니다. 문서 60여 개면 괜찮지만, 문서가 늘면 CPU 임베딩으로는 오래 걸리고 그동안 검색 결과가 비어 있게 됩니다. 그래서 Day 42의 청킹 함수를 재사용하되, 파일 해시를 payload에 저장해 **바뀐 파일만** 교체하는 `reindex.py`를 추가합니다.

```python
# rag/reindex.py — 파일 해시로 바뀐 문서만 재인덱싱 (Python 3.12)
import hashlib
import sys
import uuid
from pathlib import Path

from qdrant_client import models

from common import COLLECTION, EMBED_DIM, embed, qdrant    # Day 42
from ingest import BATCH, split_long, split_sections       # Day 42
from pii import mask                                       # Day 48
from scan import scan                                      # Day 48


def ensure_collection() -> None:
    if not qdrant.collection_exists(COLLECTION):
        qdrant.create_collection(COLLECTION, vectors_config=models.VectorParams(
            size=EMBED_DIM, distance=models.Distance.COSINE))


def indexed_hashes() -> dict[str, str]:
    """Qdrant에 저장된 source -> file_hash (Day 42 ingest.py로 넣은 포인트는 해시가 없어 한 번 재색인됨)"""
    hashes, offset = {}, None
    while True:
        points, offset = qdrant.scroll(COLLECTION, limit=1000, offset=offset,
                                       with_payload=["source", "file_hash"], with_vectors=False)
        for p in points:
            hashes[p.payload["source"]] = p.payload.get("file_hash", "")
        if offset is None:
            return hashes


def delete_source(src: str) -> None:
    qdrant.delete(COLLECTION, points_selector=models.FilterSelector(filter=models.Filter(
        must=[models.FieldCondition(key="source", match=models.MatchValue(value=src))])))


def index_file(path: Path, src: str, file_hash: str) -> int:
    text = path.read_text(encoding="utf-8")
    if findings := scan(text):
        print(f"[SKIP] {src} 의심 문구: {findings}")
        return 0
    title = next((ln[2:].strip() for ln in text.splitlines() if ln.startswith("# ")), src)
    chunks = [{"source": src, "title": title, "section": section, "text": mask(part),
               "file_hash": file_hash}
              for section, body in split_sections(text) for part in split_long(body)]
    for start in range(0, len(chunks), BATCH):
        batch = chunks[start:start + BATCH]
        vectors = embed([f"{c['title']} > {c['section']}\n{c['text']}" for c in batch])   # Day 42와 같은 형식
        qdrant.upsert(COLLECTION, points=[
            models.PointStruct(id=str(uuid.uuid5(uuid.NAMESPACE_URL, f"{src}#{start + j}")),
                               vector=v, payload=c)
            for j, (c, v) in enumerate(zip(batch, vectors))])
    return len(chunks)


def main(root: Path) -> None:
    ensure_collection()
    current = {p.relative_to(root).as_posix(): hashlib.sha256(p.read_bytes()).hexdigest()
               for p in sorted(root.rglob("*.md"))
               if not any(part.startswith(".") for part in p.relative_to(root).parts)}
    old = indexed_hashes()
    removed = old.keys() - current.keys()
    changed = [src for src, h in current.items() if old.get(src) != h]

    for src in removed:
        delete_source(src)
        print(f"[DEL] {src}")
    for src in changed:
        delete_source(src)                       # 청크 수가 줄어든 경우 옛 조각이 남지 않게
        print(f"[UPD] {src} ({index_file(root / src, src, current[src])} chunks)")
    print(f"done: {len(changed)} changed, {len(removed)} deleted, {len(current) - len(changed)} unchanged")


if __name__ == "__main__":
    main(Path(sys.argv[1]))
```

> ⚠️ 컬렉션의 벡터 차원은 **임베딩 모델과 같아야** 합니다(bge-m3는 1024, Day 42의 `EMBED_DIM`). 임베딩 모델을 바꾸면 증분이 아니라 **새 컬렉션에 전체 인덱싱 → Qdrant 컬렉션 별칭(alias)을 새 컬렉션으로 전환**하는 블루/그린 방식으로 교체하세요. 별칭을 쓰면 rag-api 설정을 바꾸지 않고 무중단으로 바꿀 수 있습니다.

### 5-2. CronJob

```yaml
# 40-rag/reindex-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: reindex
  namespace: rag
spec:
  schedule: "0 */6 * * *"
  timeZone: "Asia/Seoul"
  concurrencyPolicy: Forbid            # 이전 작업이 안 끝났으면 건너뜀
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 3600
      template:
        metadata:
          labels: { app: reindex }
        spec:
          restartPolicy: OnFailure
          initContainers:
            - name: git
              image: alpine/git:v2.54.0
              args: ["clone", "--depth", "1", "https://github.com/yeomyaloo/infra.git", "/repo"]
              volumeMounts: [{ name: repo, mountPath: /repo }]
          containers:
            - name: indexer
              image: <레지스트리>/infra-rag:0.2        # Day 43 이미지 + reindex.py·pii.py·scan.py
              command: ["python", "reindex.py", "/repo"]
              envFrom: [{ secretRef: { name: rag-api-env } }]
              resources: { requests: { cpu: 200m, memory: 256Mi } }
              volumeMounts: [{ name: repo, mountPath: /repo }]
          volumes:
            - name: repo
              emptyDir: {}
```

```bash
kubectl apply -f 40-rag/reindex-cronjob.yaml
kubectl -n rag get cronjob reindex
# NAME      SCHEDULE      TIMEZONE     SUSPEND   ACTIVE   LAST SCHEDULE   AGE
# reindex   0 */6 * * *   Asia/Seoul   False     0        <none>          5s

# 문서 하나를 고쳐 push한 뒤 즉시 실행해 보기
kubectl -n rag create job reindex-now --from=cronjob/reindex
kubectl -n rag logs -f job/reindex-now -c indexer
# [UPD] 06-ai-infra/day49-capstone-rag-chatbot.md (31 chunks)
# done: 1 changed, 0 deleted, 65 unchanged
```

> ⚠️ 7단계에서 `rag` 네임스페이스에 default deny egress를 걸었다면 `app: reindex` Pod에는 **GitHub(443)과 DNS, gateway, qdrant**로 나가는 규칙이 따로 필요합니다. NetworkPolicy는 FQDN을 지원하지 않으므로 443 전체를 허용하거나, 레포를 사내 Git 서버로 미러링하세요.

> 💡 6시간 주기 대신 "push하면 즉시"를 원한다면 GitHub Actions에서 `kubectl create job --from=cronjob/reindex`를 호출하면 됩니다 — 바로 Chapter 7의 주제입니다.

---

## 6. 외부 공개 — Cloudflare Tunnel (H03)

H03에서 서버에 직접 설치했던 `cloudflared`를 이번에는 **K8s Deployment**로 띄웁니다. 포트포워딩 없이, 서버에서 Cloudflare로 나가는 연결만으로 공개됩니다.

1. Cloudflare 대시보드(Zero Trust → Networks → Tunnels)에서 터널을 만들고 **토큰**을 복사
2. 같은 화면의 Public Hostname에 `chat.example.com` → `http://open-webui.webui.svc.cluster.local:80` 등록
3. **Access → Applications**에서 `chat.example.com`에 정책 추가(예: 회사 이메일 도메인만 One-time PIN 로그인) — Open WebUI 로그인 **앞에** 인증을 한 겹 더

```yaml
# 60-edge/cloudflared.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: cloudflared, namespace: edge }
spec:
  replicas: 2
  selector: { matchLabels: { app: cloudflared } }
  template:
    metadata: { labels: { app: cloudflared } }
    spec:
      containers:
        - name: cloudflared
          image: cloudflare/cloudflared:2026.11.0
          command: ["cloudflared", "tunnel", "--no-autoupdate", "--metrics", "0.0.0.0:2000", "run"]
          env:
            - name: TUNNEL_TOKEN
              valueFrom: { secretKeyRef: { name: tunnel-token, key: token } }
          livenessProbe:
            httpGet: { path: /ready, port: 2000 }
            failureThreshold: 1
            initialDelaySeconds: 10
            periodSeconds: 10
```

```bash
kubectl create namespace edge
kubectl -n edge create secret generic tunnel-token --from-literal=token=<터널 토큰>
kubectl apply -f 60-edge/cloudflared.yaml
kubectl -n edge logs deploy/cloudflared | grep -i "registered tunnel connection"
# 휴대폰 LTE로 https://chat.example.com → Cloudflare Access 로그인 → Open WebUI
```

| 방식 | 대상 | 장점 | 주의 |
|---|---|---|---|
| **Cloudflare Tunnel + Access** | 사내 사용자 다수, 브라우저만 있으면 됨 | 포트 0개, HTTPS·인증·DDoS 방어를 Cloudflare가 | 트래픽이 Cloudflare를 경유 (질문·답변 내용 포함) |
| **Tailscale** (H03) | 관리자·소수 팀원 | 인터넷에 아예 노출 안 됨, Kubernetes Operator로 Service를 tailnet에 노출 가능 | 사용자마다 Tailscale 클라이언트 설치 필요 |

> 💡 Grafana, Langfuse, Prometheus 같은 **관리 화면은 Tunnel로 공개하지 말고** Tailscale로만 접근하세요. 공개 표면은 챗봇 하나면 충분합니다.

---

## 7. 운영 체크리스트

| 주기 | 할 일 | 도구 |
|---|---|---|
| 매일 | 대시보드 1분 훑기: TTFT p95, 대기 요청, KV 캐시, GPU 온도, 에러율 | Grafana (Day 47) |
| 매일 | 알림 확인 — 울린 알림마다 "조치가 필요했나?" 기록 (불필요하면 규칙 수정) | Alertmanager |
| 매주 | Langfuse에서 낮은 평가·거절(`rag.rejected`)·"찾을 수 없습니다" 답변 표본 검토 → 골든 셋 보강 | Langfuse, Day 48 |
| 매주 | 재인덱싱 Job 성공 여부, `[SKIP]` 로그(인젝션 의심 문서) 검토 | `kubectl -n rag get jobs` |
| 매주 | 게이트웨이 키별 사용량·예산 소진율 | LiteLLM UI |
| 매월 | 백업: Qdrant 스냅샷(`POST /collections/<이름>/snapshots`), Open WebUI PVC, LiteLLM PostgreSQL | 스냅샷 + 오프사이트 복사 |
| 매월 | 업그레이드: **한 번에 하나씩**, 이미지 태그·차트 버전 고정, 업그레이드 전후 골든 셋 평가 비교 | Day 48 eval |
| 분기 | 키 회전(LiteLLM 마스터·가상 키, Qdrant, 터널 토큰), Access 정책·사용자 목록 정리 | - |
| 분기 | 복구 리허설: 새 클러스터에 이 디렉터리 + 백업으로 복원해 보기 | - |

> ⚠️ 가장 흔한 운영 사고는 "vLLM 이미지를 `latest`로 두었다가 재시작 시 새 버전이 받아지면서 메트릭 이름·옵션이 바뀌는" 경우입니다. **모든 이미지와 차트에 버전을 고정**하고(위 예시처럼), 바꿀 때는 의도적으로 바꾸세요.

---

## 8. 비용 비교 — 자체 호스팅 vs API

숫자는 가정이며, 실제 단가는 각 공급사 가격표와 전기요금 고지서로 다시 계산해야 합니다. 중요한 건 **계산하는 방법**입니다.

**가정 (사내 50명)**: 1인당 하루 20질문, 질문 1회당 입력 3,000 토큰(검색 문서 포함) · 출력 400 토큰, 월 22일
→ 월 입력 **6,600만 토큰**, 출력 **880만 토큰**

| 항목 | 자체 호스팅 (홈랩) | 소형 상용 API (가정 단가) | 중형 상용 API (가정 단가) |
|---|---|---|---|
| 단가 가정 | - | 입력 $0.15 / 출력 $0.60 (1M 토큰당) | 입력 $2.50 / 출력 $10.00 (1M 토큰당) |
| 토큰 비용 | 0 | 66×0.15 + 8.8×0.60 ≈ **$15** | 66×2.5 + 8.8×10 ≈ **$253** |
| 전기료 | 평균 150W × 720h ≈ 108kWh × 200원 ≈ **2.2만 원** | 0 | 0 |
| 하드웨어 감가상각 | GPU·부품 120만 원 ÷ 36개월 ≈ **3.3만 원** | 0 | 0 |
| 운영 인건비 | 월 몇 시간 (이 문서의 체크리스트) | 거의 0 | 거의 0 |
| **월 합계 (대략)** | **약 5.5만 원 (고정)** | **약 2만 원 (사용량 비례)** | **약 35만 원 (사용량 비례)** |

이 표에서 읽어야 할 것:

| 관점 | 해석 |
|---|---|
| 소형 모델끼리만 보면 | **API가 더 쌉니다.** 사용량이 적으면 고정비(전기·장비)를 이기기 어렵습니다 |
| 사용량이 늘면 | 자체 호스팅은 GPU가 감당하는 한 비용이 **거의 그대로**, API는 정비례 → 손익분기점이 생김 |
| 품질 | 1.5B 자체 모델과 상용 대형 모델의 답변 품질은 같지 않습니다. 비교는 **같은 골든 셋 점수**(Day 48)를 놓고 해야 공정합니다 |
| 돈 말고 | **데이터가 밖으로 나가지 않음**(사내 문서·질문), 오프라인 동작, 모델·버전 통제, 그리고 이 모든 걸 직접 다뤄 본 **경험** |

> 💡 현실적인 답은 대개 **하이브리드**입니다. LiteLLM 게이트웨이에 자체 모델과 외부 API를 함께 등록하고, 민감한 사내 문서 RAG는 자체 모델로, 문서와 무관한 일반 질문이나 평가용 judge는 API로 라우팅합니다. 게이트웨이의 키·예산·로그가 양쪽에 똑같이 적용됩니다.

---

## 9. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| vLLM Pod가 `Pending` (Insufficient nvidia.com/gpu) | 롤링 업데이트로 새 Pod가 GPU를 기다림 | `strategy: Recreate` (GPU 1장일 때), 또는 기존 Pod 삭제 |
| vLLM이 OOM / KV 캐시 할당 실패 | `--max-model-len`이 VRAM에 비해 큼, TEI·다른 GPU 프로세스와 경합 | `--max-model-len` 축소, `--gpu-memory-utilization` 조정, `nvidia-smi`로 다른 프로세스 확인 |
| Open WebUI 모델 목록에 docs-rag 없음 | `openaiBaseApiUrls`와 키 개수 불일치, NetworkPolicy 차단 | 키는 `;`로 URL 수만큼, `webui → rag:8000` 허용 확인 |
| 채팅 제목이 이상하고 GPU가 바쁨 | 작업 모델이 docs-rag로 설정됨 | 관리자 설정에서 작업 모델을 `qwen2.5-1.5b`로 |
| 질문은 되는데 답이 "문서에서 찾을 수 없습니다"만 | 인덱싱 안 됨, 임베딩 모델 불일치(차원·모델 변경) | `initial-index` 로그 확인, 컬렉션 벡터 차원 = 1024 확인 |
| 재인덱싱 Job이 `git clone`에서 실패 | egress 차단(DNS·443) | 5-2의 reindex egress 규칙 추가 |
| 외부에서 접속하면 502 | Public Hostname의 서비스 주소/포트 오타, cloudflared가 webui ns에 못 감 | `open-webui.webui.svc.cluster.local:80` 확인, edge → webui NetworkPolicy |
| 매일 새벽 TTFT 알림 | 재인덱싱 중 임베딩·Qdrant 부하 | CronJob 시간을 사용량 낮은 때로, TEI 자원 분리 |

---

## 10. 핵심 정리

- 부품은 이미 다 있었다 — 오늘의 일은 **조립 순서**와 **연결부(Service 주소, 키, NetworkPolicy)** 를 정확히 맞추는 것
- **아래층부터 한 층씩, 층마다 검증**: 클러스터 → GPU → 모니터링 → LLM/임베딩 → 게이트웨이 → 벡터 DB/rag-api → UI → 관측·보안 → 공개
- 게이트웨이가 있으니 백엔드(vLLM ↔ Ollama ↔ 외부 API)를 바꿔도 앱은 그대로다
- 지식 베이스는 살아 있다 → **해시 기반 증분 재인덱싱 CronJob**, 임베딩 모델 교체는 **새 컬렉션 + 별칭 전환**
- 공개는 **Cloudflare Tunnel + Access**(사용자), 관리 화면은 **Tailscale**(관리자)
- 모든 이미지·차트 **버전 고정**, 백업·키 회전·복구 리허설을 체크리스트로
- 비용은 **사용량 × 단가 vs 고정비**로 계산하고, 품질은 같은 골든 셋으로 비교한다

---

## 11. Chapter 0~6 회고

![Chapter 0~6 회고](./images/day49-series-retrospective.svg)

| Chapter | 우리가 얻은 것 | 오늘 챗봇에서 쓰인 곳 |
|---|---|---|
| **0. 홈랩** (H01~H03) | 남는 PC를 서버로, 안전한 외부 공개 | 서버 한 대, Cloudflare Tunnel |
| **1. Docker** (Day 01~07) | 이미지·볼륨·네트워크·Compose, 컨테이너 내부 원리 | 모든 구성요소가 컨테이너, rag-api 이미지 |
| **2. Kubernetes** (Day 08~20) | Pod·Deployment·Service·Secret·PVC·Ingress·Helm·Prometheus·HPA·RBAC·kubeadm | 오늘 쓴 YAML의 90% |
| **3. 가상화·네트워크** (Day 21~24) | KVM, bridge·veth·VXLAN·OVS | CNI(Calico)가 하는 일을 이해하는 바탕 |
| **4. OpenStack** (Day 25~33) | 프라이빗 클라우드 — 인증·이미지·컴퓨트·네트워크·스토리지 | GPU 패스스루 VM으로 같은 스택을 올릴 수 있는 기반 |
| **5. 종합** (Day 34~35) | K8s on OpenStack, Terraform IaC | "인프라를 코드로" — Chapter 7의 출발점 |
| **6. AI 인프라** (Day 36~49) | GPU·LLM 서빙·게이트웨이·RAG·K8s GPU·오토스케일링·관측·보안·평가 | 오늘의 챗봇 그 자체 |

`docker run hello-world`(Day 01)에서 시작해, GPU 위에서 LLM을 돌리고 내 문서로 답하는 서비스를 **직접 운영**하는 데까지 왔습니다. 각 층이 왜 필요한지, 그리고 문제가 생기면 몇 번째 층을 봐야 하는지 알게 된 것이 이 시리즈에서 얻은 가장 큰 자산입니다.

> 💡 **더 깊이 가고 싶다면 (장기 학습 방향)** — **파인튜닝·LoRA**(사내 말투·형식에 맞춘 소형 모델, vLLM의 LoRA 어댑터 서빙), **멀티 GPU 분산 추론**(텐서/파이프라인 병렬, vLLM `--tensor-parallel-size`, llm-d 같은 분산 서빙 프로젝트), **MLOps**(모델 레지스트리·실험 추적·평가 자동화, KServe·Kubeflow), **에이전트 인프라**(MCP 서버, 도구 호출 권한 설계 — OWASP Agentic Top 10). 어느 쪽이든 오늘 만든 게이트웨이·관측·평가 체계가 그대로 토대가 됩니다.

---

## 12. 셀프 체크

<details>
<summary>Q1. 배포 순서에서 모니터링(kube-prometheus-stack)을 LLM보다 먼저 설치하는 이유는?</summary>

ServiceMonitor·PrometheusRule CRD가 먼저 있어야 GPU Operator의 dcgm-exporter ServiceMonitor와 우리 매니페스트가 적용되고, 무엇보다 LLM을 처음 띄울 때의 GPU 메모리·기동 시간·에러를 **처음부터 관측**할 수 있기 때문입니다. 문제를 찾는 도구를 문제보다 먼저 준비하는 것입니다.
</details>

<details>
<summary>Q2. GPU 1장 환경에서 vLLM Deployment에 <code>strategy: Recreate</code>를 쓰는 이유는?</summary>

기본 RollingUpdate는 새 Pod를 먼저 띄운 뒤 기존 Pod를 내리는데, 새 Pod도 GPU 1장을 요구하므로 기존 Pod가 GPU를 놓기 전까지 `Pending`에서 영원히 기다립니다. Recreate는 기존 Pod를 먼저 종료하므로 (잠깐의 중단을 감수하고) 교체가 됩니다.
</details>

<details>
<summary>Q3. 재인덱싱에서 바뀐 파일의 기존 포인트를 먼저 지우고 다시 넣는 이유는?</summary>

포인트 ID를 `source#청크번호`로 만들기 때문에, 문서가 짧아져 청크 수가 10 → 7로 줄면 upsert만으로는 8~10번 청크가 **옛 내용 그대로 남습니다.** 그 조각이 검색되면 챗봇이 삭제된 내용을 답하게 되므로 source 단위로 지운 뒤 다시 넣습니다.
</details>

<details>
<summary>Q4. 사용량이 적은 팀이라면 자체 호스팅보다 API가 싼데도 자체 호스팅을 택할 이유는?</summary>

사내 문서와 질문이 **외부로 나가지 않는다**(데이터 주권·규정 준수), 오프라인/폐쇄망에서도 동작, 모델과 버전을 직접 통제, 비용이 사용량과 무관하게 고정. 반대로 품질이 최우선이면 API가 낫고, 게이트웨이로 둘을 섞는 하이브리드가 흔한 답입니다.
</details>

<details>
<summary>Q5. Grafana·Langfuse를 Cloudflare Tunnel로 공개하지 말라는 이유는?</summary>

관리 화면은 사용자 수가 적고 권한이 크며, Langfuse에는 프롬프트·답변 원문이 저장됩니다. 공개 표면을 챗봇 하나로 최소화하고, 관리 화면은 인터넷에 노출되지 않는 Tailscale 사설망으로만 접근하는 것이 안전합니다.
</details>

---

## ⏭️ 다음 예고 — Day 50. CI/CD와 GitOps — Push 배포 vs Pull 배포

오늘 우리는 `kubectl apply`와 `helm install`을 손으로 수십 번 쳐서 챗봇을 올렸습니다. 내일 서버가 죽으면 이 순서를 정확히 다시 칠 수 있을까요? 누가 언제 무엇을 바꿨는지 알 수 있을까요?

**Chapter 7. GitOps & CI/CD**(Day 50~57)에서는 오늘 만든 `homelab-rag/` 디렉터리를 **Git 저장소 하나**로 선언하고, 클러스터가 그 상태를 **스스로 따라가게** 만듭니다.

- **GitHub Actions** — rag-api 이미지 빌드·테스트·골든 셋 평가를 push마다 자동으로
- **Kustomize** — 환경별(dev/prod) 설정 차이를 깔끔하게
- **Argo CD** — Git이 곧 진실: 손으로 바꾼 것은 되돌리고, 커밋은 자동 동기화
- **Argo Rollouts** — 새 모델·프롬프트를 카나리로 조금씩, 지표가 나쁘면 자동 롤백
- **공급망 보안** — 내가 배포하는 이미지와 의존성을 믿을 수 있는가

첫날인 Day 50에서는 지금까지의 방식(**Push 배포**: 내가 클러스터에 밀어 넣기)과 GitOps의 방식(**Pull 배포**: 클러스터가 Git을 당겨 오기)을 비교하는 것부터 시작합니다. 🚀
