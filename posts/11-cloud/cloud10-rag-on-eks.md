---
title: "클라우드 10. RAG 챗봇(Day 49)을 EKS로 이전 — GPU 노드 vs Bedrock"
date: 2026-11-19T22:35:17+09:00
weight: 10
tags: ["cloud", "aws", "gpu", "rag", "eks", "llm"]
summary: "[본인 AWS 계정 · 비용 발생 가능] Day 49의 홈랩 RAG 챗봇 구성 요소를 AWS 서비스로 하나씩 대응시킬 수 있다. ① GPU 노드그룹 + vLLM 과 ② Amazon Bedrock + 같은 rag-api 를 비용·운영 부담·데이터 위치로 비교하고 고를 수 있다."
image: "./images/cloud10-options-compare.svg"
draft: false
---
> **클라우드 · AWS** | 난이도 ★★★★★ | 예상 소요 3 ~ 4시간 | 💰 예상 비용 Bedrock 경로 시간당 약 $0.35~0.5 (`infra-cloud-eks` 기본 비용 포함, 토큰 비용은 실습량 기준 수십 원) / GPU 경로는 g6.xlarge 1대만으로 시간당 약 $0.99 추가 (2026-11-19 확인, 변동 가능)

> 💸 **비용 경고 — 실습 전에 반드시 읽기**
> - 이 편은 클라우드 06의 **`infra-cloud-eks`가 떠 있는 상태**를 전제로 합니다. 클러스터 자체($0.10/시간), 노드, NAT 게이트웨이($0.059/시간 + GB당 $0.059), 로드밸런서($0.0225/시간~)가 **켜져 있는 동안 계속** 과금됩니다. 프리 티어로는 커버되지 않습니다.
> - **GPU 인스턴스(g 계열)는 프리 티어가 없고**, 서울 온디맨드 기준 g4dn.xlarge $0.647, g6.xlarge $0.9896, g5.xlarge $1.237, g6e.xlarge $2.288 / 시간입니다. **켜 둔 채 하룻밤이면 2~5만 원**입니다. 이 편의 기본 실습은 GPU를 쓰지 않습니다.
> - Bedrock은 **토큰 단위 종량제**라 실습량(질문 수십 개, 문서 인덱싱 1회)이면 수십 원 수준입니다.
> - 가격은 AWS Price List(2026-11-19 조회, `ap-northeast-2`) 기준이며 **변동 가능**합니다.
> - 끝나면 **9장 "정리"** 를 실행하고, 탭을 마칠 거라면 바로 **클라우드 11의 전체 삭제**로 넘어갑니다.

> 📌 **이 탭의 고정값** — 클라우드 01~08에서 정한 값을 그대로 씁니다.
> 리전 `ap-northeast-2` · VPC `10.20.0.0/16` · 리소스 이름 접두사 `infra-cloud-` · 공통 태그 `Project=infra-study`, `Env=lab` · CLI 프로필 `infra-admin` · EKS 클러스터 `infra-cloud-eks`
> 추가 가정: 클라우드 06에서 **EBS CSI 드라이버**와 **EKS Pod Identity Agent** 애드온을, 클라우드 07에서 AWS Load Balancer Controller를 설치했다. 없으면 7-1에서 설치합니다.

## 🎯 오늘의 목표

- Day 49의 홈랩 RAG 챗봇 구성 요소를 **AWS 서비스로 하나씩 대응**시킬 수 있다
- **① GPU 노드그룹 + vLLM** 과 **② Amazon Bedrock + 같은 rag-api** 를 비용·운영 부담·데이터 위치로 비교하고 고를 수 있다
- Bedrock의 **모델 접근 방식과 서울 리전에서 쓸 수 있는 모델**(리전 내 / 교차 리전 추론)을 확인할 수 있다
- rag-api의 **OpenAI 호환 클라이언트**를 코드 수정 없이 Bedrock에 연결한다 (LiteLLM 게이트웨이 + Pod Identity)
- 벡터 DB 선택지(Qdrant on EKS / OpenSearch Serverless / pgvector on RDS)의 차이를 안다

---

## 1. 무엇을 옮기나 — 홈랩 ↔ AWS 대응표

[Day 49](../06-ai-infra/day49-capstone-rag-chatbot.md)의 요청 흐름은 이랬습니다: 사용자 → Cloudflare Access → Open WebUI → **rag-api** → LiteLLM → (TEI 임베딩, Qdrant 검색, vLLM 생성) → 출처와 함께 답변.

| Day 49 구성 요소 | 홈랩 | ① GPU 노드 경로 | ② Bedrock 경로 (오늘 실습) |
|---|---|---|---|
| K8s 클러스터 | H01 단일 노드 kubeadm (Day 20) | `infra-cloud-eks` | `infra-cloud-eks` |
| GPU 스케줄링 | GPU Operator (Day 44) | GPU 노드그룹 + NVIDIA 디바이스 플러그인/GPU Operator | **불필요** |
| LLM 추론 | vLLM `Qwen/Qwen2.5-1.5B-Instruct` (Day 45) | vLLM 그대로 | **Bedrock** 관리형 모델 |
| 임베딩 | TEI `BAAI/bge-m3` CPU (Day 41) | TEI 그대로 | **Bedrock Titan Text Embeddings V2** |
| 게이트웨이 | LiteLLM (Day 40) | LiteLLM 그대로 | LiteLLM — **백엔드만 Bedrock으로** |
| 벡터 DB | Qdrant + local-path PVC | Qdrant + EBS gp3 PVC | Qdrant + EBS gp3 PVC (또는 OpenSearch Serverless / RDS pgvector) |
| rag-api | 레지스트리의 `infra-rag` 이미지 | **ECR** | **ECR**, 코드 변경 없음 |
| 모델 캐시 | hf-cache PVC (Day 45) | EBS PVC 또는 S3 | 불필요 |
| 자격 증명 | Secret에 키 | Secret | **Pod Identity**(키 없음) |
| 외부 공개 | Cloudflare Tunnel + Access (H03) | ALB + 인증 | ALB + 인증 (오늘은 port-forward) |
| 관측 | kube-prometheus-stack, Langfuse (Day 47) | 동일 + DCGM | 동일 + CloudWatch Bedrock 지표 (클라우드 08) |
| 보안 점검 | Day 48 가드레일 | 동일 | 동일 + 클라우드 09 GuardDuty EKS 보호 |

![RAG 챗봇 이전 — GPU 노드 vs Bedrock](./images/cloud10-options-compare.svg)

---

## 2. 선택지 ① — EKS GPU 노드그룹에서 vLLM 운영

Day 44·[Day 45](../06-ai-infra/day45-llm-on-k8s.md)에서 한 일을 클라우드 노드에서 그대로 하는 길입니다. **쿠버네티스 쪽 매니페스트는 거의 바뀌지 않습니다.** 바뀌는 것은 "GPU를 어디서 빌리느냐"입니다.

### 2-1. g 계열 인스턴스와 가격 (서울, 온디맨드 Linux, 2026-11-19 조회, 변동 가능)

| 인스턴스 | GPU | GPU 메모리 | vCPU / RAM | 시간당 | 24시간 × 30일 | 적합한 모델 (Day 36 VRAM 계산) |
|---|---|---|---|---|---|---|
| g4dn.xlarge | NVIDIA T4 ×1 | 16 GiB | 4 / 16 GiB | **$0.647** | 약 $472 | 1.5B~7B(양자화), FP8 미지원 |
| g6.xlarge | NVIDIA L4 ×1 | 24 GiB | 4 / 16 GiB | **$0.9896** | 약 $722 | 7B~8B BF16, FP8 가능 |
| g5.xlarge | NVIDIA A10G ×1 | 24 GiB | 4 / 16 GiB | **$1.237** | 약 $903 | 7B~8B BF16 |
| g6e.xlarge | NVIDIA L40S ×1 | 48 GiB | 4 / 32 GiB | **$2.288** | 약 $1,670 | 14B BF16, 32B 양자화 |

> 💡 같은 24GiB라면 서울에서는 **g6(L4)가 g5(A10G)보다 쌉니다.** Day 49의 1.5B 모델이라면 g4dn.xlarge로도 충분합니다. 스팟 인스턴스는 훨씬 싸지만 회수될 수 있어, vLLM처럼 **모델 로딩에 수 분 걸리는** 워크로드는 회수 대비(Day 45 Probe, 여러 레플리카)가 필요합니다.

### 2-2. 서비스 쿼터 — 새 계정은 GPU가 0대

EC2 온디맨드 쿼터는 **인스턴스 수가 아니라 vCPU 수** 단위이고, G 계열은 **"Running On-Demand G and VT instances"** 라는 별도 쿼터입니다. 새 계정은 이 값이 **0**인 경우가 많아, 노드그룹을 만들어도 인스턴스가 뜨지 않습니다.

```bash
aws service-quotas list-service-quotas --service-code ec2 \
  --query "Quotas[?QuotaName=='Running On-Demand G and VT instances'].[QuotaCode,Value]" --output table
# |  L-DB2E81BA |  0.0 |     ← 0이면 증가 요청 필요

aws service-quotas request-service-quota-increase --service-code ec2 \
  --quota-code L-DB2E81BA --desired-value 8          # xlarge(4 vCPU) 2대분
aws service-quotas list-requested-service-quota-change-history --service-code ec2 \
  --query 'RequestedQuotas[].[QuotaName,DesiredValue,Status]' --output table
```

승인은 수 시간~며칠 걸리고, 사용 이력이 없는 계정은 거절되거나 낮게 승인되기도 합니다. **실습 계획 며칠 전에** 요청하세요.

### 2-3. 노드그룹과 배포 (요약)

| 단계 | 내용 | 근거 |
|---|---|---|
| 노드그룹 | 관리형 노드그룹 `infra-cloud-gpu`, AMI 타입 **`AL2023_x86_64_NVIDIA`**(NVIDIA 드라이버 포함 EKS 최적화 AMI), `g6.xlarge`, desired 0~1, 테인트 `nvidia.com/gpu=true:NoSchedule` | 클라우드 06 |
| 디바이스 플러그인 | 드라이버가 AMI에 있으므로 GPU Operator는 `driver.enabled=false` 로, 또는 NVIDIA device plugin만 | Day 44 |
| 모델 캐시 | Day 45의 hf-cache PVC를 EBS gp3로. 노드가 바뀌면 다시 받으므로 S3에 미리 올려 두고 initContainer로 복사하는 방법도 있음 | Day 45 |
| 배포 | Day 45·49의 `vllm-qwen` 매니페스트 그대로 + `tolerations` | Day 45 |
| 비용 통제 | Karpenter NodePool로 GPU 노드를 요청 있을 때만, Day 46의 KEDA scale-to-zero | 클라우드 06, Day 46 |

> ⚠️ GPU 경로를 실제로 해 본다면 **노드그룹 desired를 0으로 두고 시작해서, 확인이 끝나면 즉시 0으로** 내리세요. `kubectl delete deploy`만 해서는 노드가 남아 과금됩니다.

---

## 3. 선택지 ② — Amazon Bedrock 관리형 모델 + 같은 rag-api

GPU·드라이버·모델 캐시·vLLM 버전 관리를 **전부 AWS에 넘기고**, 토큰만큼 냅니다. rag-api는 OpenAI 호환 API만 알면 되므로 **이미지는 그대로**입니다.

### 3-1. 모델 접근 — 2026-11 기준 방식

예전에는 콘솔의 **Model access** 페이지에서 모델마다 "접근 요청"을 눌러야 했지만, 지금 상용 리전에서는 방식이 바뀌었습니다(AWS 공식 문서 확인).

| 항목 | 현재 방식 |
|---|---|
| 대부분의 모델 | **기본으로 사용 가능.** 서드파티 모델은 **첫 호출 때** Bedrock이 AWS Marketplace 구독을 자동으로 시작 (최대 15분) |
| 필요한 권한 (첫 호출 1회) | 호출 주체에 `aws-marketplace:Subscribe`, `Unsubscribe`, `ViewSubscriptions` — Amazon 모델(Titan, Nova)은 Marketplace 상품이 아니라 불필요 |
| Anthropic 모델 | **Claude Opus 4.7 이전 모델**은 계정(또는 조직 관리 계정)당 1회 **사용 사례 양식(FTU)** 제출 필요. 그 이후 모델은 불필요 |
| 결제 수단 | 서드파티 모델은 Marketplace 결제가 가능한 결제 수단 필요 |
| "막고 싶을 때" | Marketplace 구독 거부만으로는 첫 호출을 못 막음 → **IAM/SCP로 `bedrock:InvokeModel*` 을 거부**해야 함 |

> 💡 이 편은 **Amazon 모델(Nova Lite, Titan Text Embeddings V2)** 만 씁니다. Marketplace 권한·양식 없이 바로 호출됩니다. 서드파티 모델을 처음 호출하면 그 모델의 EULA에 동의한 것으로 간주되니, 회사 계정이라면 먼저 확인하세요.

### 3-2. 서울(`ap-northeast-2`)에서 쓸 수 있는 모델 — 직접 확인

Bedrock은 모델마다 **리전 내(In-Region)**, **지역 교차 리전 추론(Geo, 예: `apac.` 접두사)**, **글로벌 교차 리전 추론(`global.`)** 지원 여부가 다릅니다. 2026-11-19 공식 문서 기준 서울은 대략 이렇습니다.

| 모델 | 서울 리전 내 | APAC 교차 리전 | 글로벌 | 오늘 용도 |
|---|---|---|---|---|
| **Amazon Titan Text Embeddings V2** | ✅ | - | - | 임베딩 |
| **Amazon Nova Micro / Lite / Pro** | - | ✅ | - | 답변 생성 |
| Amazon Nova 2 Lite | - | - | ✅ | (선택) |
| Anthropic Claude 최신 Haiku/Sonnet | - | - | ✅ | (선택, 데이터 위치 주의) |

목록은 자주 바뀌므로 **반드시 CLI로 직접 확인**합니다.

```bash
export AWS_REGION=ap-northeast-2
# 서울에서 리전 내로 호출 가능한 임베딩 모델
aws bedrock list-foundation-models --by-output-modality EMBEDDING \
  --query "modelSummaries[].[modelId,modelLifecycle.status]" --output table

# 서울에서 출발할 수 있는 교차 리전 추론 프로파일 (apac. / global.)
aws bedrock list-inference-profiles --type-equals SYSTEM_DEFINED \
  --query "inferenceProfileSummaries[?status=='ACTIVE'].[inferenceProfileId]" --output text | tr '\t' '\n' | grep -E 'nova|titan' 

# 이 프로파일이 요청을 실제로 보낼 수 있는 목적지 리전
aws bedrock get-inference-profile --inference-profile-identifier apac.amazon.nova-lite-v1:0 \
  --query 'models[].modelArn' --output text | tr '\t' '\n'
```

```
arn:aws:bedrock:ap-northeast-2::foundation-model/amazon.nova-lite-v1:0
arn:aws:bedrock:ap-northeast-1::foundation-model/amazon.nova-lite-v1:0
arn:aws:bedrock:ap-southeast-1::foundation-model/amazon.nova-lite-v1:0
...
```

> ⚠️ **데이터 위치** — 교차 리전 추론은 요청을 **프로파일의 목적지 리전 중 하나**로 보냅니다. `apac.` 은 아시아·태평양 리전들 안에서, `global.` 은 **전 세계 상용 리전 어디로든** 갈 수 있습니다. "사내 문서가 국외로 나가면 안 된다"는 요구가 있으면 **리전 내 모델만** 쓰거나 `get-inference-profile` 의 목적지 목록을 보고 판단해야 합니다. 접두사만 보고 판단하지 말라는 것이 AWS 문서의 권고입니다. 저장 데이터(Qdrant, S3)는 모두 서울에 남습니다.

> 💡 Bedrock은 프롬프트와 응답을 모델 학습에 쓰지 않고 모델 제공사와 공유하지 않는다고 문서화되어 있습니다. **모델 호출 로깅**은 기본 꺼짐이며, 켜면 프롬프트 원문이 내 S3/CloudWatch에 남으므로 Day 48의 PII 마스킹 이후 단계에서만 켜는 것이 좋습니다.

### 3-3. 벡터 DB 선택지

| | **Qdrant on EKS** (오늘) | **OpenSearch Serverless** (벡터 검색 컬렉션) | **pgvector on RDS PostgreSQL** |
|---|---|---|---|
| rag-api 코드 | **변경 없음** (Day 42 `qdrant-client`) | 검색 부분 교체 (`opensearch-py`, k-NN 쿼리) | 검색 부분 교체 (SQL, Day 41 pgvector 예제) |
| 운영 | Helm 업그레이드, 백업(EBS 스냅샷)을 내가 | 서버리스, 패치·확장 AWS | 패치·백업은 RDS, 인덱스 튜닝은 내가 |
| 비용 (서울, 2026-11-19, 변동 가능) | EBS gp3 GB-월 $0.0912 + 노드 자원 일부 | OCU-시간 $0.293 (인덱싱·검색 각각, 최소 용량 설정만큼 유휴에도 과금) | db.t4g.micro $0.025/시간 + 스토리지 |
| 인증 | Qdrant API 키 (Secret) | **IAM(SigV4)** — Pod Identity로 키 없이 | DB 비밀번호 또는 **RDS IAM 인증** |
| 어울리는 곳 | 학습·소규모, 홈랩과 같은 코드 유지 | 대규모, 하이브리드(키워드+벡터) 검색 | 이미 Postgres를 쓰는 팀, 메타데이터 조인 |

오늘은 **rag-api를 한 줄도 바꾸지 않는 것**이 목표이므로 Qdrant on EKS를 씁니다.

---

## 4. 비용·운영 부담·데이터 위치 비교

**가정**: Day 49 8장과 같은 사용량 — 월 입력 6,600만 토큰, 출력 880만 토큰. 서울 가격, 2026-11-19 조회, **변동 가능**.

| 항목 | ① GPU 노드 + vLLM (g6.xlarge) | ② Bedrock (Nova Lite + Titan V2) |
|---|---|---|
| 모델 추론 비용 | 24시간: 약 **$722/월** / 업무 시간(10h×22일)만: 약 **$218/월** | 66M × $0.071/1M + 8.8M × $0.284/1M ≈ **$7.2/월** |
| 임베딩 비용 | TEI를 같은 GPU 또는 CPU 노드에 | 질문 임베딩 ≈ 수십만 토큰 × $0.0246/1M ≈ **$0.1 미만** |
| 공통 고정비 (양쪽 같음) | EKS $0.10/h ≈ $73, NAT 1개 ≈ $43 + 처리량, ALB ≈ $16 + LCU, CPU 노드 | 동일 |
| 사용량이 10배가 되면 | GPU 1장이 감당하는 한 거의 그대로 | 거의 **정비례** (≈ $72) |
| 모델 품질을 올리면 | 더 큰 GPU(g6e 등) 필요 | Nova Pro 등으로 **모델 ID만** 변경 (같은 사용량 ≈ $96) |
| 운영 부담 | 드라이버·CUDA·vLLM 버전, 쿼터, 모델 캐시, 콜드 스타트, GPU 노드 스케일링 | 모델 ID·쿼터(분당 토큰) 관리 정도 |
| 콜드 스타트 | 노드 기동 + 이미지 + 모델 로딩 = 수 분 | 없음 |
| 데이터 위치 | **전부 서울 VPC 안** | 임베딩은 서울, 생성은 `apac.` 목적지 리전 중 하나 (프로파일로 확인) |
| 모델 통제 | 오픈 모델·버전 완전 통제, 파인튜닝 자유 | Bedrock 카탈로그 범위, 모델 수명 주기(Legacy/EOL) 따라감 |
| 배우는 것 | Day 44~46 그대로 클라우드에서 | 관리형 AI 서비스 연동, IAM 기반 호출 |

> 💡 결론은 Day 49 8장과 같습니다. **사용량이 적으면 종량제(②)가 압도적으로 싸고**, 사용량이 GPU를 꽉 채울 만큼 많거나 데이터를 한 리전·한 VPC에 묶어야 하면 ①이 맞습니다. 그리고 **LiteLLM 게이트웨이 덕분에 둘 다 등록해 두고 라우팅**하는 하이브리드가 가능합니다.

---

## 5. rag-api를 Bedrock에 연결하는 세 가지 방법

rag-api는 Day 42의 `common.py` 에서 `OpenAI(base_url=LLM_BASE_URL, api_key=LLM_API_KEY)` 로 **OpenAI 호환 API**를 부릅니다. Bedrock 쪽에서 이것을 받는 방법은 세 가지입니다(2026-11-19 공식 문서 확인).

| 방법 | 엔드포인트 | 인증 | 장점 | 한계 |
|---|---|---|---|---|
| A. Bedrock 자체 OpenAI 호환 API (`bedrock-runtime`) | `https://bedrock-runtime.ap-northeast-2.amazonaws.com/openai/v1` | **SigV4** 또는 **Bedrock API 키**(Bearer) | 중간 계층 없음 | **모델마다 지원 여부가 다름**(모델 카드의 "Chat Completions" 확인), `GET /models` 미지원, **임베딩 API는 이 경로로 쓰지 않음**, OpenAI SDK는 SigV4를 못 하므로 사실상 **API 키** 필요 |
| B. `bedrock-mantle` 호환 엔드포인트 | `https://bedrock-mantle.<리전>.api.aws/v1` | Bedrock API 키 또는 AWS 자격 증명 | `/models` 지원, 일부 모델 전용 | AWS가 "`bedrock-runtime` 에 없는 경우에만" 쓰라고 권고, 리전·모델 제한 |
| **C. LiteLLM 게이트웨이 경유** (오늘) | 클러스터 안 `http://litellm.gateway.svc.cluster.local:4000/v1` | LiteLLM → Bedrock은 **boto3 기본 자격 증명 체인**(Pod Identity) | rag-api·Open WebUI **무변경**, 채팅·임베딩 모두, 가상 키·예산·Langfuse 그대로, GPU 경로와 하이브리드 | 게이트웨이를 내가 운영 |

> ⚠️ **Bedrock API 키**는 편하지만 **장기 비밀**입니다(단기 키도 있음). 클라우드 01·09의 원칙대로, 클러스터 안 워크로드는 **IAM 역할(Pod Identity)** 로 호출하는 것이 맞습니다. 방법 C는 키를 Secret에 넣지 않습니다.

Day 49에서 "게이트웨이를 둔 덕분에 rag-api와 Open WebUI는 한 줄도 바뀌지 않는다"고 했던 바로 그 지점입니다. 바뀌는 것은 **LiteLLM의 `model_list` 두 항목과 rag-api Secret의 `LLM_MODEL` 값 하나**뿐입니다.

![Bedrock 경로 아키텍처 — infra-cloud-eks 안의 RAG 스택](./images/cloud10-bedrock-architecture.svg)

---

## 6. 실습 ① — Bedrock 호출 확인 (CLI)

실습 환경: Ubuntu 24.04(WSL2) + AWS CLI v2 + kubectl + Helm 3 + Docker. SSO로 로그인합니다.

```bash
aws sso login --profile lab
export AWS_PROFILE=lab AWS_REGION=ap-northeast-2
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws eks update-kubeconfig --name infra-cloud-eks
```

```bash
# 생성 모델 (APAC 교차 리전 추론 프로파일)
aws bedrock-runtime converse --model-id apac.amazon.nova-lite-v1:0 \
  --messages '[{"role":"user","content":[{"text":"EKS를 한 문장으로 설명해줘."}]}]' \
  --inference-config '{"maxTokens":200}' \
  --query 'output.message.content[0].text' --output text

# 임베딩 모델 (서울 리전 내) — 1024차원
aws bedrock-runtime invoke-model --model-id amazon.titan-embed-text-v2:0 \
  --body '{"inputText":"쿠버네티스 Pod란?","dimensions":1024,"normalize":true}' \
  --cli-binary-format raw-in-base64-out /tmp/emb.json
python3 -c "import json;d=json.load(open('/tmp/emb.json'));print(len(d['embedding']), d['inputTextTokenCount'])"
# 1024 7
```

> 💡 `AccessDeniedException` 이 나면 SSO 권한 세트에 `bedrock:InvokeModel` 이 있는지, `ValidationException ... on-demand throughput isn't supported` 가 나면 **모델 ID 대신 추론 프로파일 ID**(`apac.`)를 써야 하는 모델입니다.

---

## 7. 실습 ② — infra-cloud-eks에 Bedrock 경로로 배포

### 7-1. 사전 애드온 확인

```bash
aws eks list-addons --cluster-name infra-cloud-eks --output text
# eks-pod-identity-agent, aws-ebs-csi-driver 가 없으면:
aws eks create-addon --cluster-name infra-cloud-eks --addon-name eks-pod-identity-agent
# EBS CSI 드라이버는 자체 IAM 권한이 필요 — 클라우드 06의 방식(Pod Identity 연결)으로 설치
```

EBS gp3 StorageClass (클라우드 06에서 기본 SC를 만들었다면 건너뜁니다):

```yaml
# gp3-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
  annotations: { storageclass.kubernetes.io/is-default-class: "true" }
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
  tagSpecification_1: "Project=infra-study"
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
allowVolumeExpansion: true
```

```bash
kubectl apply -f gp3-sc.yaml
```

> 💡 `encrypted: "true"` 는 EBS 볼륨을 KMS로 암호화합니다(`kmsKeyId` 를 주면 클라우드 09 방식의 고객 관리형 키). `tagSpecification_1` 로 태그를 붙여 두면 클라우드 11에서 **남은 볼륨을 태그로 찾을 수** 있습니다.

### 7-2. LiteLLM용 IAM 역할과 Pod Identity

LiteLLM Pod(`gateway` 네임스페이스, ServiceAccount `litellm`)만 Bedrock의 **두 모델만** 호출할 수 있게 합니다.

```bash
cat > /tmp/trust.json <<'EOF'
{ "Version": "2012-10-17",
  "Statement": [{ "Effect": "Allow",
    "Principal": { "Service": "pods.eks.amazonaws.com" },
    "Action": ["sts:AssumeRole", "sts:TagSession"] }] }
EOF

cat > /tmp/bedrock-invoke.json <<EOF
{ "Version": "2012-10-17",
  "Statement": [
    { "Sid": "InvokeNovaLiteViaApacProfile", "Effect": "Allow",
      "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"],
      "Resource": [
        "arn:aws:bedrock:ap-northeast-2:${ACCOUNT_ID}:inference-profile/apac.amazon.nova-lite-v1:0",
        "arn:aws:bedrock:*::foundation-model/amazon.nova-lite-v1:0" ] },
    { "Sid": "InvokeTitanEmbedInRegion", "Effect": "Allow",
      "Action": "bedrock:InvokeModel",
      "Resource": "arn:aws:bedrock:ap-northeast-2::foundation-model/amazon.titan-embed-text-v2:0" }
  ] }
EOF

aws iam create-role --role-name infra-cloud-litellm-bedrock \
  --assume-role-policy-document file:///tmp/trust.json --tags Key=Project,Value=infra-study
aws iam put-role-policy --role-name infra-cloud-litellm-bedrock \
  --policy-name bedrock-invoke --policy-document file:///tmp/bedrock-invoke.json

kubectl create namespace gateway
kubectl -n gateway create serviceaccount litellm
aws eks create-pod-identity-association --cluster-name infra-cloud-eks \
  --namespace gateway --service-account litellm \
  --role-arn arn:aws:iam::${ACCOUNT_ID}:role/infra-cloud-litellm-bedrock --tags Project=infra-study
```

| 정책 포인트 | 이유 |
|---|---|
| 추론 프로파일 ARN **과** `foundation-model/*` 리전 와일드카드 | 교차 리전 추론은 프로파일과 **목적지 리전의 모델** 양쪽에 권한이 있어야 함 |
| `Converse` 액션이 없다? | Converse API는 `bedrock:InvokeModel`(스트리밍은 `InvokeModelWithResponseStream`) 권한으로 호출됨 |
| `Resource: "*"` 금지 | 클라우드 09의 Access Analyzer 미사용 권한 결과가 깨끗하게 나오도록, 처음부터 두 모델만 |

> 💡 [해킹 09](../hacking/hack09-cloud-iam-mistakes.md)에서 본 대로, 노드 역할에 Bedrock 권한을 주면 **노드의 모든 Pod**가 Bedrock을 부를 수 있습니다. Pod Identity는 **ServiceAccount 단위**로 역할을 줍니다. 클라우드 09에서 IMDSv2 홉 제한을 1로 두면 Pod가 노드 역할을 빌려 쓰는 길까지 막힙니다.

### 7-3. rag-api 이미지를 ECR로

```bash
aws ecr create-repository --repository-name infra-cloud/infra-rag \
  --image-scanning-configuration scanOnPush=true --image-tag-mutability IMMUTABLE \
  --tags Key=Project,Value=infra-study
REG=${ACCOUNT_ID}.dkr.ecr.ap-northeast-2.amazonaws.com
aws ecr get-login-password | docker login --username AWS --password-stdin $REG

# Day 49에서 빌드한 이미지 (Qdrant api_key 수정 + reindex.py 포함본)
docker tag infra-rag:0.2 $REG/infra-cloud/infra-rag:0.2
docker push $REG/infra-cloud/infra-rag:0.2
```

> ⚠️ 노드가 **arm64(Graviton)** 라면 `docker buildx build --platform linux/amd64,linux/arm64` 로 다시 빌드합니다(Day 51 멀티 아키텍처). 이미지가 amd64뿐이면 Pod가 `exec format error` 로 죽습니다.

### 7-4. LiteLLM — 백엔드만 Bedrock으로

Day 49의 `30-gateway/litellm-config.yaml` 에서 `model_list` 만 바꿉니다.

```yaml
# 30-gateway/litellm-config.yaml (EKS 판)
model_list:
  - model_name: chat
    litellm_params:
      model: bedrock/apac.amazon.nova-lite-v1:0      # 교차 리전 추론 프로파일 ID
      aws_region_name: ap-northeast-2
      input_cost_per_token: 0.000000071              # 서울 $0.071/1M (2026-11-19, 변동 가능)
      output_cost_per_token: 0.000000284             # 서울 $0.284/1M
  - model_name: embed                                # Day 49와 같은 이름 → rag-api 설정 그대로
    litellm_params:
      model: bedrock/amazon.titan-embed-text-v2:0   # 기본 출력 1024차원 (256/512도 가능)
      aws_region_name: ap-northeast-2

litellm_settings:
  success_callback: ["langfuse"]          # Day 47 — Langfuse를 안 쓰면 이 두 줄 삭제

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  database_url: os.environ/DATABASE_URL
```

자격 증명 항목(`aws_access_key_id` 등)을 **넣지 않았다**는 점이 핵심입니다. LiteLLM은 boto3의 기본 자격 증명 체인을 쓰고, Pod Identity Agent가 Pod에 주입한 컨테이너 자격 증명 엔드포인트(`AWS_CONTAINER_CREDENTIALS_FULL_URI`)를 boto3가 자동으로 사용합니다.

```bash
kubectl -n gateway create configmap litellm-config --from-file=config.yaml=30-gateway/litellm-config.yaml
kubectl -n gateway create secret generic litellm-env \
  --from-literal=LITELLM_MASTER_KEY=sk-master-$(openssl rand -hex 16) \
  --from-literal=LITELLM_SALT_KEY=$(openssl rand -hex 32) \
  --from-literal=DATABASE_URL=postgresql://litellm:CHANGEME@litellm-db:5432/litellm
kubectl apply -f 30-gateway/litellm.yaml          # Day 49 매니페스트 그대로 (PostgreSQL PVC는 gp3로 생성됨)
# LiteLLM Deployment가 위 ServiceAccount를 쓰게
kubectl -n gateway patch deployment litellm -p '{"spec":{"template":{"spec":{"serviceAccountName":"litellm"}}}}'
kubectl -n gateway rollout status deploy/litellm

# Pod에 자격 증명이 주입됐는지
kubectl -n gateway exec deploy/litellm -- env | grep AWS_CONTAINER
# AWS_CONTAINER_CREDENTIALS_FULL_URI=http://169.254.170.23/v1/credentials
# AWS_CONTAINER_AUTHORIZATION_TOKEN_FILE=/var/run/secrets/pods.eks.amazonaws.com/serviceaccount/eks-pod-identity-token
```

가상 키를 발급하고 두 모델을 시험합니다(Day 48·49와 같은 방법).

```bash
kubectl -n gateway port-forward svc/litellm 4000:4000 &
MASTER=$(kubectl -n gateway get secret litellm-env -o jsonpath='{.data.LITELLM_MASTER_KEY}' | base64 -d)
RAG_KEY=$(curl -s localhost:4000/key/generate -H "Authorization: Bearer $MASTER" -H 'Content-Type: application/json' \
  -d '{"key_alias":"rag-api","models":["chat","embed"],"max_budget":2,"budget_duration":"30d","rpm_limit":60}' | jq -r .key)

curl -s localhost:4000/v1/chat/completions -H "Authorization: Bearer $RAG_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"chat","messages":[{"role":"user","content":"안녕? 한 문장으로."}]}' | jq -r '.choices[0].message.content'
curl -s localhost:4000/v1/embeddings -H "Authorization: Bearer $RAG_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"embed","input":"테스트"}' | jq '.data[0].embedding | length'
# 1024
```

> 💡 `max_budget: 2` 는 이 가상 키로 쓸 수 있는 금액 상한($2)입니다. 위에서 넣은 `input_cost_per_token` 으로 LiteLLM이 계산합니다. **AWS Budgets(클라우드 01)와 별도로, 앱 단위 예산**이 한 겹 더 생깁니다.

### 7-5. Qdrant와 rag-api

```bash
kubectl create namespace rag
helm repo add qdrant https://qdrant.github.io/qdrant-helm && helm repo update
cat > /tmp/qdrant-values.yaml <<EOF
apiKey: "$(openssl rand -hex 24)"
persistence:
  size: 10Gi
  storageClassName: gp3
EOF
helm install qdrant qdrant/qdrant -n rag --version 1.20.1 -f /tmp/qdrant-values.yaml
QDRANT_KEY=$(grep apiKey /tmp/qdrant-values.yaml | cut -d'"' -f2)

kubectl -n rag create secret generic rag-api-env \
  --from-literal=LLM_BASE_URL=http://litellm.gateway.svc.cluster.local:4000/v1 \
  --from-literal=LLM_API_KEY=$RAG_KEY \
  --from-literal=LLM_MODEL=chat \
  --from-literal=EMBED_BASE_URL=http://litellm.gateway.svc.cluster.local:4000/v1 \
  --from-literal=EMBED_API_KEY=$RAG_KEY \
  --from-literal=EMBED_MODEL=embed \
  --from-literal=EMBED_DIM=1024 \
  --from-literal=COLLECTION=infra_docs_titan \
  --from-literal=QDRANT_URL=http://qdrant.rag.svc.cluster.local:6333 \
  --from-literal=QDRANT_API_KEY=$QDRANT_KEY
```

Day 49와 다른 값은 **`LLM_MODEL=chat`** 과 **`COLLECTION=infra_docs_titan`** 두 개뿐입니다.

> ⚠️ **차원이 같아도 벡터는 호환되지 않습니다.** bge-m3와 Titan V2는 둘 다 1024차원이지만 **서로 다른 의미 공간**입니다. 홈랩 Qdrant의 스냅샷을 그대로 옮겨 오면 검색이 엉망이 됩니다. 반드시 **새 컬렉션에 전체 재인덱싱**합니다(Day 49 5-1의 블루/그린 원칙).

Day 49의 `40-rag/rag-api.yaml` 과 `reindex-cronjob.yaml` 에서 **이미지 주소만** ECR로 바꿔 적용합니다.

```bash
sed "s#image: .*infra-rag:.*#image: ${REG}/infra-cloud/infra-rag:0.2#" 40-rag/rag-api.yaml | kubectl apply -n rag -f -
sed "s#image: <레지스트리>/infra-rag:0.2#image: ${REG}/infra-cloud/infra-rag:0.2#" 40-rag/reindex-cronjob.yaml | kubectl apply -n rag -f -

kubectl -n rag create job initial-index --from=cronjob/reindex
kubectl -n rag logs -f job/initial-index -c indexer
# [NEW] 00-homelab/homelab01-ubuntu-server.md (23 chunks)
# ...
# done: 60 changed, 0 deleted, 0 unchanged
```

> 💡 노드가 ECR에서 이미지를 받으려면 노드 역할에 `AmazonEC2ContainerRegistryReadOnly`(또는 그에 준하는 정책)가 필요합니다. EKS 관리형 노드그룹의 기본 노드 역할에는 보통 들어 있습니다.

### 7-6. 동작 확인

```bash
kubectl -n rag port-forward svc/rag-api 8000:8000 &
curl -s localhost:8000/v1/chat/completions -H 'Content-Type: application/json' \
  -d '{"model":"docs-rag","messages":[{"role":"user","content":"Day 45에서 vLLM에 /dev/shm을 왜 붙였어?"}]}' \
  | jq -r '.choices[0].message.content'
```

```
vLLM은 텐서 병렬·프로세스 간 통신에 공유 메모리를 쓰는데, 컨테이너 기본 /dev/shm(64MB)은 너무 작아서 ...

**출처**
- `06-ai-infra/day45-llm-on-k8s.md` — 왜 `/dev/shm`이 필요한가
```

Open WebUI는 Day 49 6단계의 Helm values 그대로 `webui` 네임스페이스에 설치하고 `kubectl -n webui port-forward svc/open-webui 8080:80` 으로 접속합니다. 모델 목록에 `docs-rag` 와 `chat` 이 보이면 성공입니다.

> ⚠️ **외부 공개는 오늘 하지 않습니다.** ALB로 열려면 클라우드 07의 AWS Load Balancer Controller + ACM 인증서(HTTPS) + 인증(ALB의 OIDC/Cognito 인증, 또는 Day 73 Keycloak)을 함께 붙여야 합니다. 인증 없이 Open WebUI를 인터넷에 여는 것은 [Day 66](../09-security/day66-threat-modeling-4c.md) STRIDE 표의 S(위장)·I(정보 노출)를 그대로 허용하는 일입니다. 잠깐 시험만 할 거라면 Ingress에 `alb.ingress.kubernetes.io/inbound-cidrs: <내 공인 IP>/32` 로 출발지를 좁힙니다.

### 7-7. 클라우드 08·09와 연결

| 확인할 것 | 어디서 |
|---|---|
| Bedrock 호출 수·지연·토큰 | CloudWatch 지표 네임스페이스 `AWS/Bedrock` (`Invocations`, `InvocationLatency`, `InputTokenCount`, `OutputTokenCount`) — 클라우드 08 대시보드에 추가 |
| 앱별 사용량·비용 | LiteLLM 키별 spend, Langfuse 트레이스 (Day 47) |
| 누가 Bedrock을 불렀나 | CloudTrail (`infra-cloud-litellm-bedrock` 역할 세션) — 클라우드 09 |
| 과한 권한 | Access Analyzer 미사용 권한 — 오늘 만든 역할이 두 모델만 쓰는지 |
| 클러스터 이상 행동 | GuardDuty EKS Protection 결과 |

---

## 8. (선택) 선택지 ①을 직접 해 보고 싶다면 — 체크리스트

| ☐ | 할 일 |
|---|---|
| ☐ | 2-2의 쿼터 승인 확인 (`L-DB2E81BA` ≥ 4) |
| ☐ | 관리형 노드그룹 `infra-cloud-gpu`: `AL2023_x86_64_NVIDIA`, `g4dn.xlarge` 또는 `g6.xlarge`, min 0 / desired 1 / max 1, 테인트 `nvidia.com/gpu=true:NoSchedule`, 태그 `Project=infra-study` |
| ☐ | `kubectl get nodes -l eks.amazonaws.com/nodegroup=infra-cloud-gpu -o jsonpath='{..allocatable.nvidia\.com/gpu}'` 가 `1` |
| ☐ | Day 45 `vllm-qwen` 매니페스트에 `tolerations` 추가, hf-cache PVC `storageClassName: gp3` |
| ☐ | LiteLLM `model_list` 에 Day 49의 `qwen2.5-1.5b` 항목을 **추가**(Bedrock `chat` 과 공존 = 하이브리드) |
| ☐ | 끝나면 **노드그룹 desired 0** 또는 노드그룹 삭제, PVC 삭제 → EBS 볼륨 남았는지 확인 |

---

## 9. 정리

```bash
# Kubernetes (PVC가 지워져야 EBS 볼륨도 지워짐: reclaimPolicy Delete)
helm -n rag uninstall qdrant
kubectl delete namespace rag webui gateway          # PVC 포함 삭제
kubectl get pv | grep -E 'rag/|gateway/|webui/' || echo "PV 없음"

# Pod Identity 연결, IAM 역할
ASSOC=$(aws eks list-pod-identity-associations --cluster-name infra-cloud-eks --namespace gateway \
  --query 'associations[0].associationId' --output text)
aws eks delete-pod-identity-association --cluster-name infra-cloud-eks --association-id "$ASSOC"
aws iam delete-role-policy --role-name infra-cloud-litellm-bedrock --policy-name bedrock-invoke
aws iam delete-role --role-name infra-cloud-litellm-bedrock

# ECR (이미지 포함 삭제)
aws ecr delete-repository --repository-name infra-cloud/infra-rag --force

# 남은 EBS 볼륨 확인 (태그로)
aws ec2 describe-volumes --filters Name=tag:Project,Values=infra-study Name=status,Values=available \
  --query 'Volumes[].[VolumeId,Size,CreateTime]' --output table
```

> 💡 탭을 여기서 마칠 거라면 위 정리 대신 **클라우드 11의 전체 삭제 순서**를 따르세요. `infra-cloud-eks`와 VPC까지 한 번에 지우고, 숨은 비용까지 확인합니다.

---

## 10. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| LiteLLM 로그 `NoCredentialsError` / `Unable to locate credentials` | Deployment가 `litellm` SA를 안 씀, Pod Identity Agent 없음, 연결 후 Pod 재시작 안 함 | 7-4 patch, `aws eks list-addons`, `kubectl rollout restart` |
| `AccessDeniedException ... inference-profile/apac...` | 정책에 프로파일 ARN 없음 또는 목적지 리전 모델 ARN 없음 | 7-2 정책의 두 Resource 모두 필요 |
| `ValidationException: Invocation of model ID ... with on-demand throughput isn't supported` | 프로파일 전용 모델을 모델 ID로 호출 | `apac.`/`global.` 프로파일 ID 사용 |
| `ThrottlingException` | 분당 요청·토큰 쿼터 초과 | 재인덱싱 동시성 낮추기, Service Quotas에서 Bedrock 쿼터 확인·증가 요청 |
| 검색 결과가 엉뚱함 | 옛 bge-m3 컬렉션을 그대로 사용 | 새 컬렉션 `infra_docs_titan` 에 전체 재인덱싱 |
| Qdrant Pod `Pending`, PVC `Pending` | EBS CSI 드라이버 없음 또는 SC 이름 불일치 | 애드온 설치, `kubectl get sc`, `WaitForFirstConsumer` 이므로 Pod 스케줄 후 생성됨 |
| rag-api `ImagePullBackOff` | ECR 주소 오타, 노드 역할에 ECR 읽기 권한 없음, 아키텍처 불일치 | `kubectl describe pod`, 7-3 경고 |
| GPU 노드그룹 생성 후 노드 0대 | G 계열 vCPU 쿼터 0 | 2-2 쿼터 증가 |

---

## 11. 핵심 정리

- Day 49의 구조는 그대로, **GPU를 빌리는 방법**만 고르면 된다: ① GPU 노드그룹 + vLLM, ② Bedrock 관리형 모델.
- 서울 가격(2026-11-19, 변동 가능): g6.xlarge **$0.9896/h** vs Nova Lite **입력 $0.071 / 출력 $0.284 (1M 토큰)**. 사용량이 적으면 ②가 수십 배 싸다.
- G 계열은 **vCPU 단위 쿼터("Running On-Demand G and VT instances")** 가 새 계정에서 0인 경우가 많다 — 미리 요청.
- Bedrock 모델 접근은 이제 **첫 호출 시 자동**(서드파티는 Marketplace 권한, 옛 Anthropic 모델은 FTU 양식). 막으려면 IAM/SCP로 `InvokeModel` 거부.
- 서울에서 Titan Embeddings V2는 **리전 내**, Nova는 **`apac.` 교차 리전 추론**. 데이터 위치는 `get-inference-profile` 의 목적지로 판단.
- Bedrock에도 OpenAI 호환 Chat Completions가 있지만 모델별 지원·임베딩·인증(SigV4/API 키) 제약이 있어, **LiteLLM 경유 + Pod Identity** 가 rag-api 무변경·키 없는 구성이다.
- **차원이 같아도 임베딩 모델이 바뀌면 전체 재인덱싱.**

---

## 12. 셀프 체크

<details>
<summary>Q1. 새 계정에서 g6.xlarge 노드그룹을 만들었는데 노드가 하나도 뜨지 않는다. 가장 먼저 의심할 것은?</summary>

EC2 서비스 쿼터 **"Running On-Demand G and VT instances"** 입니다. 이 쿼터는 인스턴스 수가 아니라 **vCPU 수** 기준이고 새 계정은 0인 경우가 많습니다. g6.xlarge 1대에 4 vCPU가 필요하므로 4 이상으로 증가 요청을 해야 합니다. 노드그룹의 Health 이슈나 Auto Scaling 활동 기록에 쿼터 관련 오류가 남습니다.
</details>

<details>
<summary>Q2. <code>apac.amazon.nova-lite-v1:0</code> 을 호출하는 IAM 정책에 프로파일 ARN만 넣었더니 거부된다. 왜인가?</summary>

교차 리전 추론은 **추론 프로파일**과, 프로파일이 요청을 보낼 **각 목적지 리전의 파운데이션 모델** 양쪽에 대한 권한을 확인합니다. 그래서 `inference-profile/apac.amazon.nova-lite-v1:0` 과 `arn:aws:bedrock:*::foundation-model/amazon.nova-lite-v1:0` 을 함께 허용해야 합니다. SCP로 목적지 리전 중 하나를 막아 둬도 요청이 실패할 수 있습니다.
</details>

<details>
<summary>Q3. bge-m3와 Titan Text Embeddings V2는 둘 다 1024차원인데 왜 기존 Qdrant 컬렉션을 그대로 쓰면 안 되나?</summary>

차원 수는 벡터의 "길이"일 뿐이고, 각 축이 무엇을 뜻하는지(의미 공간)는 모델마다 다릅니다. 질문은 Titan으로, 문서는 bge-m3로 임베딩하면 코사인 유사도가 의미 없는 값이 되어 검색이 무너집니다. 임베딩 모델을 바꾸면 **새 컬렉션에 전체 재인덱싱 후 전환**(Day 49의 블루/그린, 별칭 전환)합니다.
</details>

<details>
<summary>Q4. Bedrock API 키를 rag-api Secret에 넣고 OpenAI 호환 엔드포인트를 직접 부르는 방법 대신 LiteLLM + Pod Identity를 고른 이유 세 가지는?</summary>

① **장기 비밀이 없다** — Pod Identity가 단기 자격 증명을 자동 교체해 주고, Secret 유출 위험이 사라진다. ② **임베딩까지 한 경로** — Bedrock의 OpenAI 호환 경로는 모델별 지원이 다르고 임베딩은 이 경로로 쓰지 않으므로, rag-api가 두 백엔드를 따로 알아야 한다. LiteLLM은 채팅·임베딩을 같은 OpenAI 형식으로 감싼다. ③ **기존 운영 기능 유지** — Day 48·49의 가상 키, 예산, rpm 제한, Langfuse 트레이스, 그리고 GPU 모델과의 하이브리드 라우팅을 그대로 쓴다.
</details>

<details>
<summary>Q5. "사내 문서는 국외로 나가면 안 된다"는 요구가 생겼다. 오늘 구성에서 무엇을 바꿔야 하나?</summary>

저장 데이터(Qdrant EBS, ECR, 로그)는 이미 서울에 있습니다. 문제는 **생성 단계의 `apac.` 교차 리전 추론**입니다. 질문과 검색된 문서 조각이 프롬프트로 다른 APAC 리전에 갈 수 있으므로, ① 서울 **리전 내**로 제공되는 생성 모델로 바꾸거나(`list-foundation-models` 로 확인), ② 선택지 ①(서울 VPC 안의 GPU 노드 + vLLM)로 생성만 옮깁니다. LiteLLM에 두 백엔드를 등록하고 민감 문서 질의만 ②로 보내는 하이브리드도 가능합니다. `global.` 프로파일은 쓰지 않습니다.
</details>

---

## ⏭️ 다음 예고 — 클라우드 11. FinOps와 정리 — Cost Explorer, 태깅, 리소스 완전 삭제

클라우드 탭의 마지막입니다. 지금까지 `Project=infra-study` 태그를 붙여 온 이유를 **Cost Explorer와 비용 할당 태그**로 확인하고, **Budgets 액션·Compute Optimizer·Savings Plans** 개념을 정리합니다. 그리고 NAT·EKS·로드밸런서·GuardDuty 엔드포인트처럼 **숨어서 새는 비용**을 찾은 뒤, Kubernetes 리소스 → Helm → `terraform destroy` → 남은 ENI·스냅샷·로그 그룹·ECR까지 **탭 전체를 0원으로 되돌리는 순서**를 따라갑니다.
