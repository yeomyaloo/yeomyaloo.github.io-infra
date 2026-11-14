---
title: "Day 40. LLM 게이트웨이 — LiteLLM으로 모델 통합·키·사용량 관리"
date: 2026-11-15T03:14:35+09:00
weight: 40
tags: ["ai-infra", "llm", "llm-gateway"]
summary: "모델 서버 앞에 게이트웨이가 왜 필요한지 설명할 수 있다. LiteLLM Proxy를 Docker Compose로 띄우고 Ollama·vLLM을 하나의 OpenAI 호환 엔드포인트로 묶을 수 있다."
image: "./images/day40-gateway-architecture.svg"
draft: false
---
> **Chapter 6 · AI 인프라** | 난이도 ★★★☆☆ | 예상 소요 2시간

## 🎯 오늘의 목표

- 모델 서버 앞에 **게이트웨이**가 왜 필요한지 설명할 수 있다
- LiteLLM Proxy를 Docker Compose로 띄우고 Ollama·vLLM을 **하나의 OpenAI 호환 엔드포인트**로 묶을 수 있다
- `config.yaml`의 `model_list`, `router_settings`로 **로드밸런싱과 fallback**을 설정할 수 있다
- **가상 키(virtual key)**를 발급해 모델 접근, 예산, rate limit을 사용자별로 제한할 수 있다
- 사용량(spend) 로그가 어디에 쌓이는지, 왜 Postgres가 필요한지 안다

---

## 1. 왜 게이트웨이인가?

Day 38과 39를 거치며 우리 홈랩에는 엔드포인트가 이렇게 생겼습니다.

| 서버 | 주소 | 모델 | 인증 |
|---|---|---|---|
| Ollama | `http://<서버>:11434/v1` | `qwen2.5:1.5b` 등 여러 개 | 없음 |
| vLLM | `http://<서버>:8000/v1` | `qwen2.5-1.5b` | `--api-key` 하나 (또는 없음) |
| 외부 API (선택) | 클라우드 업체 주소 | 상용 모델 | 업체 API 키 |

팀원 다섯 명이 이걸 쓰기 시작하면 곧 이런 질문이 나옵니다.

- "어느 주소로 붙어야 해요? 모델 이름은 뭐였죠?" → **엔드포인트와 이름이 제각각**
- "누가 GPU를 이렇게 많이 쓰는 거죠?" → **사용자별 사용량을 모름**
- "인턴 실습용 키에 상용 API 비용이 수십만 원 나왔어요" → **예산·한도가 없음**
- "vLLM 서버 재시작하는 동안 서비스가 다 멈췄어요" → **장애 시 우회 경로가 없음**
- "외부 API 키가 소스 코드마다 박혀 있어요" → **비밀 관리가 분산**

![LLM 게이트웨이가 필요한 이유](./images/day40-gateway-architecture.svg)

**LLM 게이트웨이**는 이 문제들을 한 곳에서 처리하는 **리버스 프록시**입니다. Day 15의 Ingress가 HTTP 트래픽을 서비스로 라우팅했다면, LLM 게이트웨이는 **모델 이름**을 보고 백엔드 모델 서버로 라우팅하면서 인증·과금·한도를 함께 처리합니다.

| 기능 | 게이트웨이가 하는 일 | 비유 |
|---|---|---|
| 단일 엔드포인트 | 모든 모델을 `/v1/chat/completions` 하나로 | Ingress의 단일 진입점 |
| 모델 별칭 | 클라이언트는 `qwen`만 알면 됨. 실제 서버는 게이트웨이 설정에서 교체 | DNS / Service 이름 |
| 가상 키 | 사용자·팀별 키 발급, 허용 모델 지정 | K8s RBAC (Day 19) |
| 예산·rate limit | 키별 최대 비용, 분당 요청·토큰 수 제한 | ResourceQuota |
| 로드밸런싱 | 같은 모델을 서빙하는 여러 서버에 분산 | Service의 엔드포인트 분산 |
| Fallback | 실패하면 다른 모델로 자동 재시도 | 이중화 |
| 사용량 로그 | 요청마다 토큰 수·비용·지연 시간 기록 | 액세스 로그 + 과금 |

---

## 2. LiteLLM 구조

LiteLLM은 두 가지 얼굴이 있습니다.

| 형태 | 설명 |
|---|---|
| **Python SDK** | `litellm.completion(model="ollama_chat/...")`처럼 100개 이상 업체 API를 OpenAI 형식으로 통일해 호출하는 라이브러리 |
| **Proxy Server (AI Gateway)** | 위 SDK를 감싼 **독립 서버**. 오늘 쓰는 것. Docker 이미지로 배포 |

Proxy의 구성 요소는 다음과 같습니다.

| 구성 요소 | 필수 여부 | 역할 |
|---|---|---|
| LiteLLM Proxy 컨테이너 | 필수 | 4000번 포트에서 OpenAI 호환 API + 관리 API + Admin UI(`/ui`) 제공 |
| `config.yaml` | 필수 | 모델 목록, 라우팅, 전역 설정 |
| **Postgres** | 가상 키·예산·사용량 로그를 쓰려면 **필수** | 키, 팀, 사용자, spend 로그 저장. 없으면 마스터 키 하나로만 동작 |
| Redis | 선택 | Proxy를 **여러 대** 띄울 때 rate limit 카운터·쿨다운 상태 공유 |

> 💡 공식 문서 기준으로 DB 없이 가상 키를 쓰면 `No connected db` 에러가 납니다. 즉 "모델 묶기"만 할 거면 Proxy 하나로 충분하지만, **"누가 얼마나 썼나"**를 관리하려면 Postgres가 반드시 필요합니다. Day 06에서 Compose로 Web + DB를 묶은 것과 같은 구성입니다.

### 핵심 환경 변수

| 변수 | 의미 |
|---|---|
| `LITELLM_MASTER_KEY` | 관리자 키. 관리 API 호출과 Admin UI 로그인 비밀번호. **`sk-`로 시작**해야 함 |
| `LITELLM_SALT_KEY` | UI/DB에 저장하는 업체 API 키를 암호화하는 키. **한 번 정하면 바꾸지 말 것** (바꾸면 저장된 키를 복호화 못 함) |
| `DATABASE_URL` | `postgresql://user:pass@host:5432/db` 형식 |

---

## 3. 라우팅: 로드밸런싱과 Fallback

![LiteLLM 라우팅과 fallback](./images/day40-routing-fallback.svg)

LiteLLM의 라우팅은 **모델 그룹** 개념으로 이해하면 쉽습니다.

- `model_list`에서 **같은 `model_name`을 가진 항목들**이 하나의 모델 그룹이 됩니다
- 클라이언트가 `"model": "qwen"`을 요청하면 그룹 안의 **배포(deployment)** 중 하나로 보냅니다 → **로드밸런싱**
- 한 배포가 연속으로 실패하면 `cooldown_time` 동안 그룹에서 빼 둡니다
- 그룹 전체가 실패하면 `fallbacks`에 적힌 **다른 그룹**으로 넘어갑니다 → **fallback**

| `routing_strategy` | 동작 |
|---|---|
| `simple-shuffle` (기본, 운영 권장) | 무작위 분산. `weight`나 `rpm`/`tpm`을 주면 그 비율대로 |
| `least-busy` | 진행 중인 요청이 가장 적은 배포로 |
| `latency-based-routing` | 응답이 빠른 배포 우선 |
| `usage-based-routing-v2` | 분당 토큰 사용량 기준 (Redis 권장) |
| `cost-based-routing` | 가장 싼 배포 우선 |

---

## 4. 실습 환경

| 항목 | 기준 |
|---|---|
| OS | Ubuntu 24.04 LTS (Day 37~39의 GPU 서버) |
| 백엔드 1 | Ollama (Day 38) — 11434 포트, `qwen2.5:1.5b` 모델 |
| 백엔드 2 | vLLM (Day 39, 7-3의 compose) — 8000 포트, 모델명 `qwen2.5-1.5b`, API 키 없음 |
| 게이트웨이 | `ghcr.io/berriai/litellm:v1.109.1` (2026-11 기준 최신 안정 릴리스) + `postgres:17` |
| 클라이언트 | curl, Python 3.12 + `openai` |

```bash
# 백엔드가 살아 있는지 먼저 확인
ollama pull qwen2.5:1.5b            # Ollama를 컨테이너로 띄웠다면: docker exec -it ollama ollama pull qwen2.5:1.5b
curl -s http://localhost:11434/v1/models | jq '.data[].id'
curl -s http://localhost:8000/v1/models  | jq '.data[].id'
```

> ⚠️ **GPU가 없다면?** vLLM 없이 **Ollama 하나(CPU 모드)**만으로도 오늘 실습의 90%를 할 수 있습니다. 아래 `config.yaml`에서 vLLM 항목을 지우고, 로드밸런싱 실습은 같은 Ollama를 두 번 등록(`weight`만 다르게)해 동작만 확인하면 됩니다. 외부 API 키가 있다면 fallback 대상으로 붙여 보는 것도 좋습니다.

> 💡 Ollama를 컨테이너가 아닌 **호스트에 직접 설치**했다면 기본적으로 `127.0.0.1`에만 열려 있어 다른 컨테이너가 접근하지 못합니다. `sudo systemctl edit ollama`로 `Environment="OLLAMA_HOST=0.0.0.0:11434"`를 추가하고 재시작하세요. (외부 노출은 방화벽으로 막을 것)

---

## 5. 실습 — LiteLLM Proxy 띄우기

### 5-1. 디렉터리와 비밀 값

```bash
mkdir -p ~/llm-gateway && cd ~/llm-gateway

cat > .env <<EOF
LITELLM_MASTER_KEY=sk-$(openssl rand -hex 24)
LITELLM_SALT_KEY=sk-$(openssl rand -hex 24)
POSTGRES_PASSWORD=$(openssl rand -hex 16)
# 외부 API를 쓸 때만 (선택)
# OPENAI_API_KEY=sk-...
EOF
chmod 600 .env
cat .env | grep MASTER      # 이 값이 관리자 키
```

### 5-2. `config.yaml`

```yaml
# ~/llm-gateway/config.yaml
model_list:
  # ① 개별 이름: 특정 서버를 지정하고 싶을 때
  - model_name: qwen-vllm
    litellm_params:
      model: hosted_vllm/qwen2.5-1.5b            # hosted_vllm/<vLLM의 served-model-name>
      api_base: http://host.docker.internal:8000/v1
      # 자체 서버는 비용이 0이라 예산 실습을 위해 가상의 단가를 붙임 (USD/토큰)
      input_cost_per_token: 0.000001
      output_cost_per_token: 0.000002

  - model_name: qwen-ollama
    litellm_params:
      model: ollama_chat/qwen2.5:1.5b             # 채팅은 ollama_chat/ 접두사 권장
      api_base: http://host.docker.internal:11434
      input_cost_per_token: 0.000001
      output_cost_per_token: 0.000002

  # ② 그룹 이름 "qwen": 같은 model_name 두 개 → 로드밸런싱
  - model_name: qwen
    litellm_params:
      model: hosted_vllm/qwen2.5-1.5b
      api_base: http://host.docker.internal:8000/v1
      weight: 3                                   # vLLM이 처리량이 높으니 3:1로
      input_cost_per_token: 0.000001
      output_cost_per_token: 0.000002
  - model_name: qwen
    litellm_params:
      model: ollama_chat/qwen2.5:1.5b
      api_base: http://host.docker.internal:11434
      weight: 1
      input_cost_per_token: 0.000001
      output_cost_per_token: 0.000002

  # ③ (선택) 외부 API — 키가 있을 때만 주석 해제. 모델 ID는 업체 문서에서 확인
  # - model_name: cloud
  #   litellm_params:
  #     model: openai/<업체의-모델-ID>
  #     api_key: os.environ/OPENAI_API_KEY

router_settings:
  routing_strategy: simple-shuffle
  allowed_fails: 3          # 쿨다운 시간 안에 3번 넘게 실패하면
  cooldown_time: 30         # 30초 동안 그 배포를 그룹에서 제외
  fallbacks:
    - qwen-vllm: ["qwen-ollama"]       # vLLM이 죽으면 Ollama로

litellm_settings:
  num_retries: 2            # 같은 그룹 안에서 재시도
  request_timeout: 60       # 초

general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  database_url: os.environ/DATABASE_URL
```

| 키 | 의미 |
|---|---|
| `model_name` | 클라이언트가 요청에 쓰는 **공개 이름** |
| `litellm_params.model` | `<업체 접두사>/<실제 모델 ID>`. `hosted_vllm/`, `ollama_chat/`, `openai/` 등 |
| `api_base` | 백엔드 주소. vLLM은 `/v1`까지, Ollama는 호스트:포트까지 |
| `os.environ/VAR` | 값을 파일에 쓰지 않고 **환경 변수에서 읽기** — 비밀은 항상 이렇게 |
| `weight` / `rpm` / `tpm` | 로드밸런싱 비율·배포별 한도 |

### 5-3. `compose.yaml`

```yaml
# ~/llm-gateway/compose.yaml
services:
  litellm:
    image: ghcr.io/berriai/litellm:v1.109.1
    container_name: litellm
    command: ["--config", "/app/config.yaml", "--port", "4000"]
    ports:
      - "4000:4000"
    volumes:
      - ./config.yaml:/app/config.yaml:ro
    env_file: .env
    environment:
      DATABASE_URL: postgresql://litellm:${POSTGRES_PASSWORD}@db:5432/litellm
      STORE_MODEL_IN_DB: "True"          # UI에서 추가한 모델도 DB에 저장
    extra_hosts:
      - "host.docker.internal:host-gateway"   # 리눅스에서 호스트의 Ollama/vLLM 접근
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:17
    container_name: litellm-db
    environment:
      POSTGRES_USER: litellm
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: litellm
    volumes:
      - litellm-pg:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U litellm -d litellm"]
      interval: 5s
      retries: 10
    restart: unless-stopped

volumes:
  litellm-pg:
```

```bash
docker compose up -d
docker compose logs -f litellm
```

```
...
LiteLLM: Proxy initialized with Config, Set models:
    qwen-vllm
    qwen-ollama
    qwen
    qwen
INFO:     Uvicorn running on http://0.0.0.0:4000 (Press CTRL+C to quit)
```

> 💡 첫 기동 때는 Postgres에 테이블을 만드는 **DB 마이그레이션**이 돌기 때문에 30초~1분 정도 걸립니다. `curl http://localhost:4000/health/liveliness`가 응답하면 준비 완료입니다.

> 💡 이미지 태그는 Day 02에서 배운 대로 `main-stable`·`latest` 같은 움직이는 태그 대신 **릴리스 태그로 고정**했습니다. 공식 문서는 `docker.litellm.ai/berriai/litellm` 레지스트리도 안내합니다.

### 5-4. 하나의 엔드포인트로 호출

```bash
source .env

# 게이트웨이가 아는 모델 목록 — 서버가 몇 대든 이름만 보임
curl -s http://localhost:4000/v1/models -H "Authorization: Bearer $LITELLM_MASTER_KEY" | jq '.data[].id'
```

```
"qwen-vllm"
"qwen-ollama"
"qwen"
```

```bash
curl -s http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen-ollama", "messages": [{"role": "user", "content": "안녕! 한 문장으로 자기소개 해줘."}]}' \
  | jq -r '.choices[0].message.content'
```

같은 요청에서 `"model"`만 `qwen-vllm`으로 바꾸면 vLLM이 답합니다. **클라이언트 코드는 그대로, 백엔드만 바뀝니다.**

### 5-5. 로드밸런싱 확인

LiteLLM은 응답 헤더에 **실제로 어느 배포가 처리했는지**를 붙여 줍니다.

```bash
for i in $(seq 1 12); do
  curl -s -o /dev/null -D - http://localhost:4000/v1/chat/completions \
    -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
    -H "Content-Type: application/json" \
    -d '{"model": "qwen", "max_tokens": 5, "messages": [{"role": "user", "content": "hi"}]}' \
  | grep -i "^x-litellm-model-api-base"
done | sort | uniq -c
```

```
      9 x-litellm-model-api-base: http://host.docker.internal:8000/v1
      3 x-litellm-model-api-base: http://host.docker.internal:11434
```

`weight: 3 / 1`이 대략 그대로 반영됩니다. (무작위 분산이라 매번 조금씩 다릅니다)

### 5-6. Fallback 확인

```bash
# vLLM을 일부러 내린다 (Day 39 디렉터리에서)
docker stop vllm

curl -s -D - -o /tmp/out.json http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen-vllm", "messages": [{"role": "user", "content": "ping"}]}' \
  | grep -i -E "^x-litellm-(model-api-base|attempted-fallbacks)"
jq -r '.choices[0].message.content' /tmp/out.json
```

```
x-litellm-model-api-base: http://host.docker.internal:11434
x-litellm-attempted-fallbacks: 1
```

`qwen-vllm`을 요청했지만 실패하자 `qwen-ollama`가 대신 답했습니다. 사용자는 에러 대신 응답을 받습니다. 이번엔 `"model": "qwen"`으로 보내 보세요. 그룹 안에서 vLLM 배포가 쿨다운에 들어가고 **Ollama 배포로만** 분산되는 것을 볼 수 있습니다.

```bash
docker start vllm    # 복구. 30초(cooldown_time) 후 다시 분산 대상에 포함
```

> ⚠️ 문서에 나오는 `mock_testing_fallbacks` 플래그는 최근 버전부터 **Proxy 요청에서는 무시**됩니다. 프록시의 fallback은 오늘처럼 테스트 환경에서 백엔드를 실제로 내려서 확인하세요.

---

## 6. 실습 — 가상 키, 예산, Rate limit

이제 마스터 키는 관리자만 쓰고, 사용자에게는 **가상 키**를 나눠 줍니다.

### 6-1. 키 발급

```bash
# 인턴용: 로컬 모델만, 하루 예산 $0.01, 분당 요청 5회
curl -s http://localhost:4000/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "intern-kim",
    "models": ["qwen", "qwen-ollama"],
    "max_budget": 0.01,
    "budget_duration": "1d",
    "rpm_limit": 5,
    "tpm_limit": 20000,
    "duration": "30d",
    "metadata": {"team": "infra-study"}
  }' | jq '{key, key_alias, models, max_budget, expires}'
```

```json
{
  "key": "sk-Xy3...q9A",
  "key_alias": "intern-kim",
  "models": ["qwen", "qwen-ollama"],
  "max_budget": 0.01,
  "expires": "2026-12-15T..."
}
```

| 필드 | 의미 |
|---|---|
| `models` | 이 키로 호출할 수 있는 모델 이름 목록 |
| `max_budget` | 최대 비용(USD). 넘으면 요청 거부 |
| `budget_duration` | 예산 초기화 주기 (`30s`, `1h`, `1d`, `30d` …) |
| `rpm_limit` / `tpm_limit` | 분당 요청 수 / 분당 토큰 수 |
| `duration` | 키 자체의 유효 기간 |
| `metadata` | 자유 형식 메모 (팀, 용도 등) |

### 6-2. 사용자 입장에서 호출

```python
# client.py — 사용자는 게이트웨이 주소와 자기 키만 안다
from openai import OpenAI

client = OpenAI(base_url="http://localhost:4000/v1", api_key="sk-Xy3...q9A")  # 발급받은 가상 키

resp = client.chat.completions.create(
    model="qwen",
    messages=[{"role": "user", "content": "Docker 볼륨을 한 줄로 설명해줘."}],
)
print(resp.choices[0].message.content)
print(resp.usage)
```

### 6-3. 허용되지 않은 모델, rate limit, 예산

```bash
KEY=sk-Xy3...q9A   # 발급받은 키

# ① 허용 목록에 없는 모델 → 401 계열 에러
curl -s http://localhost:4000/v1/chat/completions -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen-vllm", "messages": [{"role": "user", "content": "hi"}]}' | jq '.error.message'

# ② 1분 안에 8번 연속 호출 → 6번째부터 429
for i in $(seq 1 8); do
  curl -s -o /dev/null -w "%{http_code}\n" http://localhost:4000/v1/chat/completions \
    -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
    -d '{"model": "qwen", "max_tokens": 5, "messages": [{"role": "user", "content": "hi"}]}'
done
```

```
200
200
200
200
200
429
429
429
```

예산은 6-1에서 붙인 가상의 단가 기준으로 쌓입니다. 긴 답변을 여러 번 요청해 $0.01을 넘기면 `Budget has been exceeded` 류의 에러로 거부됩니다. (`budget_duration`이 지나면 초기화)

### 6-4. 사용량 확인

```bash
# 키별 누적 비용
curl -s "http://localhost:4000/key/info?key=$KEY" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" | jq '.info | {key_alias, spend, max_budget, budget_reset_at}'

# 요청 단위 로그 (모델, 토큰 수, 비용, 시작·종료 시각)
curl -s "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  | jq '.[:3][] | {model, model_group, total_tokens, spend, startTime}'
```

이 기록은 모두 Postgres에 쌓입니다.

```bash
docker exec -it litellm-db psql -U litellm -d litellm -c '\dt' | grep -i -E "spend|verificationtoken"
```

브라우저로 `http://<서버>:4000/ui`에 접속해 `admin` / 마스터 키로 로그인하면 **Virtual Keys**, **Usage**, **Logs** 화면에서 같은 정보를 그래프로 볼 수 있습니다.

> 💡 Day 38의 **Open WebUI**도 OpenAI 호환 연결 주소를 `http://<서버>:4000/v1` + 가상 키로 바꾸면 게이트웨이 뒤로 들어옵니다. 이제 채팅 UI 사용량도 키별로 집계됩니다.

### 6-5. 정리

```bash
docker compose down          # 컨테이너만 삭제 (DB 볼륨은 유지)
# docker compose down -v     # DB 볼륨까지 삭제 — 키·사용량 기록이 전부 사라짐
```

---

## 7. 운영 시 고려 사항

| 주제 | 권장 |
|---|---|
| 고가용성 | Proxy를 2대 이상 + 앞단 로드밸런서. 이때 rate limit·쿨다운 공유를 위해 **Redis** 추가 |
| 비밀 관리 | 업체 키는 `os.environ/`로만. K8s에서는 Secret (Day 13) |
| 마스터 키 | 사람에게 나눠 주지 말 것. 자동화도 관리용 가상 키를 따로 발급 |
| `LITELLM_SALT_KEY` | 백업 필수. 잃어버리면 DB에 저장된 업체 키를 못 읽음 |
| 모니터링 | Prometheus 연동(Day 17) — Day 47에서 TTFT·토큰 처리량과 함께 다룸 |
| K8s 배포 | 공식 Helm 차트 제공 (Day 16) — Day 45·49에서 vLLM과 함께 배포 |

> 💡 LiteLLM 말고도 Kong AI Gateway, Envoy AI Gateway(Day 15의 Gateway API 기반) 같은 선택지가 있습니다. 개념(단일 엔드포인트, 키, 한도, fallback)은 모두 같습니다.

---

## 8. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| `Connection refused` / `Cannot connect to host host.docker.internal` | 리눅스에서 `host.docker.internal` 미정의, 또는 백엔드가 127.0.0.1에만 열림 | compose에 `extra_hosts: host-gateway` 추가, Ollama는 `OLLAMA_HOST=0.0.0.0:11434` |
| vLLM 호출 시 404 | `api_base`에 `/v1` 누락, 또는 모델명이 `--served-model-name`과 다름 | `http://...:8000/v1` + `hosted_vllm/<served-model-name>` |
| 가상 키 발급 시 `No connected db` | `DATABASE_URL` 미설정 또는 DB 연결 실패 | compose의 `DATABASE_URL`과 Postgres healthcheck 확인 |
| `/key/generate`가 401 | 마스터 키가 아닌 키로 호출, 또는 `sk-`로 시작하지 않는 마스터 키 | `.env`의 `LITELLM_MASTER_KEY` 확인 후 재기동 |
| 사용량 spend가 계속 0 | 자체 호스팅 모델은 단가 정보가 없음 | `input_cost_per_token` / `output_cost_per_token` 지정 |
| rate limit이 Proxy 여러 대에서 제각각 | 카운터가 프로세스 메모리에만 있음 | Redis 연결로 상태 공유 |
| `LITELLM_SALT_KEY`를 바꾼 뒤 UI에서 추가한 모델이 전부 실패 | 저장된 업체 키 복호화 불가 | 원래 salt 키 복원. 없다면 업체 키 재등록 |

---

## 9. 핵심 정리

- LLM 게이트웨이 = 모델 서버 앞의 **리버스 프록시** + 인증 + 과금 + 한도 + 라우팅
- LiteLLM Proxy는 `config.yaml`의 `model_list`로 Ollama·vLLM·외부 API를 **하나의 OpenAI 호환 엔드포인트**로 통합
- 같은 `model_name` = 하나의 모델 그룹 → **로드밸런싱**. 그룹이 실패하면 `fallbacks`의 다른 그룹으로
- **가상 키**로 사용자별 허용 모델, `max_budget`, `budget_duration`, `rpm_limit`, `tpm_limit` 지정
- 가상 키·예산·사용량 로그에는 **Postgres 필수**, 다중 인스턴스 한도 공유에는 Redis
- 비밀 값은 `os.environ/`로 주입하고, 마스터 키와 salt 키는 따로 안전하게 보관

---

## 10. 셀프 체크

<details>
<summary>Q1. 앱 코드에서 vLLM 주소를 직접 쓰는 대신 게이트웨이를 거치면 얻는 이점 세 가지는?</summary>

① 모델 서버를 교체·증설해도 클라이언트 코드는 그대로(모델 별칭) ② 사용자·팀별 키로 인증, 예산, rate limit 적용 ③ 장애 시 fallback·로드밸런싱, 그리고 요청 단위 사용량 기록.
</details>

<details>
<summary>Q2. <code>model_list</code>에 <code>model_name: qwen</code> 항목이 두 개 있다. 무슨 의미인가?</summary>

두 배포가 하나의 모델 그룹 `qwen`을 이룹니다. `qwen` 요청은 `routing_strategy`(기본 `simple-shuffle`)와 `weight`에 따라 둘 중 하나로 분산되고, 하나가 실패하면 그룹 안의 다른 배포로 재시도합니다.
</details>

<details>
<summary>Q3. 로드밸런싱과 fallback의 차이는?</summary>

로드밸런싱은 **같은 모델 그룹 안**의 여러 배포에 평소 트래픽을 나누는 것이고, fallback은 그룹 전체가 실패했을 때 **다른 모델 그룹**으로 넘기는 것입니다. 예: `qwen-vllm` 실패 → `qwen-ollama`.
</details>

<details>
<summary>Q4. Postgres 없이 LiteLLM Proxy를 띄우면 무엇을 할 수 있고, 무엇을 못 하나?</summary>

마스터 키 하나로 모델 통합, 라우팅, fallback은 가능합니다. 하지만 가상 키 발급, 키별 예산·사용량 추적, spend 로그 저장은 DB가 필요해 쓸 수 없습니다.
</details>

<details>
<summary>Q5. 자체 GPU에서 돌리는 모델의 spend가 0으로 나온다. 예산 제한을 적용하려면?</summary>

`litellm_params`에 `input_cost_per_token`, `output_cost_per_token`으로 내부 단가(예: GPU 시간 비용을 토큰당으로 환산)를 지정합니다. 또는 비용 대신 `tpm_limit`/`rpm_limit`으로 사용량을 제한합니다.
</details>

---

## ⏭️ 다음 예고 — Day 41. 임베딩과 벡터 DB — Qdrant / pgvector

LLM은 학습하지 않은 우리 회사 문서를 모릅니다. 문서를 **숫자 벡터(임베딩)**로 바꿔 저장해 두고, 질문과 **의미가 가까운** 문서를 찾아 LLM에게 건네주면 어떨까요? RAG의 첫 단추인 임베딩 모델과 벡터 DB(Qdrant, pgvector), 그리고 빠른 검색을 가능하게 하는 HNSW 인덱스를 다룹니다.
