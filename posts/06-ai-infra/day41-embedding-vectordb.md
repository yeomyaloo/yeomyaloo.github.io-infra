---
title: "Day 41. 임베딩과 벡터 DB — Qdrant / pgvector"
date: 2026-11-15T03:14:35+09:00
weight: 41
tags: ["ai-infra", "llm", "vectordb"]
summary: "임베딩이 무엇이고, 왜 키워드 검색 대신 의미 검색이 가능한지 설명할 수 있다. 코사인 유사도를 계산하고 해석할 수 있다. 한국어를 지원하는 임베딩 모델을 고르고 Ollama로 서빙할 수 있다."
image: "./images/day41-embedding-cosine.svg"
draft: false
---
> **Chapter 6 · AI 인프라** | 난이도 ★★★☆☆ | 예상 소요 2 ~ 2.5시간

## 🎯 오늘의 목표

- **임베딩**이 무엇이고, 왜 키워드 검색 대신 **의미 검색**이 가능한지 설명할 수 있다
- **코사인 유사도**를 계산하고 해석할 수 있다
- 한국어를 지원하는 임베딩 모델을 고르고 Ollama로 서빙할 수 있다
- Qdrant를 Docker로 띄워 **컬렉션 생성 → upsert → 검색 → 필터**를 할 수 있다
- 벡터 인덱스 **HNSW**의 원리를 그림으로 설명하고, Qdrant와 pgvector를 비교할 수 있다

---

## 1. 왜 임베딩인가?

사내 위키에서 "**컨테이너가 자꾸 죽어요**"라고 검색했다고 해 봅시다. 정답 문서의 제목은 "**OOMKilled 발생 시 메모리 limit 조정 방법**"입니다.

| 검색 방식 | 동작 | 결과 |
|---|---|---|
| 키워드 검색 (`LIKE '%죽어요%'`, BM25) | 같은 **단어**가 있는지 비교 | 겹치는 단어가 없어 **못 찾음** |
| 의미 검색 (임베딩) | 문장의 **의미**를 숫자 벡터로 바꿔 가까운 것을 찾음 | "죽는다 ≈ 종료된다 ≈ OOMKilled"를 알아서 **찾음** |

Day 42에서 만들 **RAG**는 "질문과 관련된 문서를 찾아 LLM에게 함께 건네주는" 구조입니다. 이때 "관련된 문서 찾기"를 담당하는 것이 오늘의 **임베딩 + 벡터 DB**입니다. LLM이 모르는 우리 회사 문서를 아는 척하게 만드는 첫 단추죠.

---

## 2. 임베딩과 코사인 유사도

![임베딩과 코사인 유사도](./images/day41-embedding-cosine.svg)

**임베딩(embedding)**은 텍스트를 고정 길이의 실수 벡터로 바꾼 것입니다.

```
"고양이가 소파에서 잔다"  →  [0.021, -0.113, 0.087, ..., 0.004]   (1024개의 숫자)
```

- 임베딩 모델은 **의미가 비슷한 문장일수록 벡터가 비슷한 방향**을 가리키도록 학습돼 있습니다
- 차원 수(예: 1024)는 모델마다 고정. **같은 컬렉션에는 같은 모델의 벡터만** 넣어야 합니다
- LLM(생성 모델)과 달리 텍스트를 만들지 않고 **벡터 하나만 출력** → 작고 빠름

### 2-1. 코사인 유사도

두 벡터가 **얼마나 같은 방향인지**를 재는 지표입니다.

```
              A · B            (각 성분을 곱해서 더한 값)
cos(θ) = ─────────────── = ─────────────────────────────
           |A| × |B|          (두 벡터 길이의 곱)
```

| 값 | 의미 |
|---|---|
| 1 | 완전히 같은 방향 (의미가 거의 같음) |
| 0 | 직각 (관련 없음) |
| -1 | 정반대 방향 |

> 💡 실제 임베딩 모델의 점수는 대부분 0.3~0.9 사이에 몰려 있습니다. "0.7이면 관련 있음" 같은 **절대 기준은 모델마다 다르니**, 순위(top-k)로 보는 것이 기본입니다.

벡터를 미리 길이 1로 **정규화**해 두면 `|A| × |B| = 1`이 되어 코사인 유사도 = **내적(dot product)**이 됩니다. 벡터 DB가 `Cosine`을 선택하면 내부적으로 정규화 후 내적을 쓰는 이유입니다.

| 거리 함수 | Qdrant | pgvector 연산자 | 언제 |
|---|---|---|---|
| 코사인 | `Cosine` | `<=>` (코사인 **거리** = 1 − 유사도) | 텍스트 임베딩의 기본값 |
| 내적 | `Dot` | `<#>` (음의 내적) | 정규화된 벡터, 추천 시스템 |
| 유클리드 | `Euclid` | `<->` | 이미지·좌표 등 거리 자체가 의미 있을 때 |

---

## 3. 임베딩 모델 고르기

한국어 문서를 다룬다면 **다국어(multilingual) 또는 한국어 특화** 모델이어야 합니다. 영어 전용 모델에 한국어를 넣으면 점수가 엉망이 됩니다.

| 모델 (Hugging Face ID) | 차원 | 최대 입력 | 라이선스 | 특징 |
|---|---|---|---|---|
| [`BAAI/bge-m3`](https://huggingface.co/BAAI/bge-m3) | 1024 | 8192 토큰 | MIT | 100개 이상 언어, 한국어 성능 좋음. **Ollama `bge-m3`로 바로 사용**. 오늘의 기본 |
| [`nlpai-lab/KURE-v1`](https://huggingface.co/nlpai-lab/KURE-v1) | 1024 | 8192 토큰 | MIT | bge-m3를 **한국어 검색용으로 추가 학습**. 한국어 위주면 후보 |
| [`Qwen/Qwen3-Embedding-0.6B`](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B) | 1024 | 32K 토큰 | Apache-2.0 | 다국어, 긴 문서. Ollama `qwen3-embedding:0.6b` |
| [`intfloat/multilingual-e5-large`](https://huggingface.co/intfloat/multilingual-e5-large) | 1024 | 512 토큰 | MIT | 입력 앞에 `query: ` / `passage: ` 접두사를 붙여야 함 |
| [`sentence-transformers/all-MiniLM-L6-v2`](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) | 384 | 256 토큰 | Apache-2.0 | 매우 작고 빠름. **영어 전용** — 한국어엔 부적합 |

고를 때 볼 것:

1. **언어** — 한국어 지원 여부가 1순위
2. **차원** — 클수록 표현력↑, 저장 공간·검색 비용↑ (1024차원 float32 = 벡터당 4KB)
3. **최대 입력 길이** — 청크 크기(Day 42)를 이 안에 맞춰야 함
4. **우리 데이터로 평가** — 공개 벤치마크(MTEB 등) 순위보다 내 문서 20~30개로 직접 검색해 보는 것이 정확 (Day 48)

### 임베딩 서빙 방법

| 방법 | 장점 | 오늘 |
|---|---|---|
| **Ollama** `/api/embed` | Day 38에서 이미 설치. CPU에서도 충분히 빠름 | ✅ 사용 |
| HF **TEI** (Text Embeddings Inference) | 임베딩 전용 고성능 서버, CPU/GPU 이미지 제공 | 대안으로 소개 |
| vLLM | 생성 모델과 같은 방식으로 임베딩 모델 서빙 | Day 45에서 K8s와 함께 |

> 💡 **GPU가 없어도 오늘 실습은 전부 CPU로 됩니다.** bge-m3는 약 0.6B 파라미터라 CPU에서도 문장 수십 개를 몇 초 안에 임베딩합니다. 수십만 문서를 한 번에 색인할 때만 GPU(또는 클라우드 GPU 임대)가 의미가 있습니다.

---

## 4. 왜 벡터 DB인가? — HNSW 인덱스

문서가 100개라면 질문 벡터와 100개를 전부 비교(**전수 비교, brute force**)해도 됩니다. 1천만 개라면? 질문 하나에 1천만 번 × 1024차원 계산은 너무 느립니다.

벡터 DB는 **근사 최근접 이웃(ANN, Approximate Nearest Neighbor)** 인덱스로 "100% 정확하진 않지만 거의 정확한 top-k를 훨씬 빠르게" 찾습니다. 가장 널리 쓰이는 것이 **HNSW**(Hierarchical Navigable Small World)입니다.

![HNSW 인덱스](./images/day41-hnsw.svg)

- 벡터들을 **가까운 이웃끼리 연결한 그래프**로 만들고, 이를 **여러 층**으로 쌓습니다
- 위층일수록 노드가 적고 연결이 **멀리** 뻗음 (고속도로), 아래층일수록 촘촘함 (골목길)
- 검색은 맨 위층의 진입점에서 시작해 질문과 **더 가까운 이웃으로 계속 이동**(greedy)하고, 더 못 가면 한 층 내려갑니다
- 맨 아래층(모든 벡터 포함)에서 주변을 넓게 훑어 top-k를 반환

> 💡 지하철 노선도를 떠올리면 쉽습니다. 급행으로 목적지 근처까지 빠르게 간 뒤, 완행으로 갈아타 정확한 역에 내립니다.

| 파라미터 | 의미 | Qdrant 기본값 | pgvector 기본값 |
|---|---|---|---|
| `m` | 노드 하나가 갖는 이웃(간선) 수 | 16 | 16 |
| `ef_construct` / `ef_construction` | 인덱스를 만들 때 살펴볼 후보 수 | 100 | 64 |
| `hnsw_ef` / `hnsw.ef_search` | **검색할 때** 살펴볼 후보 수. 클수록 정확, 느림 | 요청마다 `params.hnsw_ef`로 지정 | 40 |

값을 키우면 **정확도(recall)↑, 메모리·속도↓**. 이것이 ANN의 기본 트레이드오프입니다.

---

## 5. Qdrant 기본 개념

![벡터 검색 흐름](./images/day41-vector-search-flow.svg)

| Qdrant | RDB 비유 | 설명 |
|---|---|---|
| **Collection** | 테이블 | 같은 차원·거리 함수를 쓰는 벡터들의 묶음 |
| **Point** | 행 | `id` + `vector` + `payload` 한 세트 |
| **Vector** | (특수 컬럼) | 임베딩. 검색 대상 |
| **Payload** | 일반 컬럼 (JSON) | 원문, 카테고리, 작성일 등. **필터 조건**으로 사용 |
| Payload index | 인덱스 | 자주 필터하는 필드에 생성 |

포트는 **6333 (REST + 웹 대시보드)**, **6334 (gRPC)** 두 개입니다.

---

## 6. 실습 환경

| 항목 | 기준 |
|---|---|
| OS | Ubuntu 24.04 LTS (GPU 불필요, Docker Desktop도 가능) |
| 임베딩 | Ollama (Day 38) + `bge-m3` |
| 벡터 DB | `qdrant/qdrant:v1.20.1`, `pgvector/pgvector:0.8.7-pg17` (2026-11 기준) |
| Python | 3.12, `qdrant-client`, `requests`, `psycopg[binary]`, `fastapi`, `uvicorn` |

```bash
mkdir -p ~/vectordb && cd ~/vectordb
python3.12 -m venv .venv && source .venv/bin/activate
pip install qdrant-client requests "psycopg[binary]" fastapi uvicorn
```

---

## 7. 실습 — 임베딩 만들기

### 7-1. Ollama로 bge-m3 임베딩

```bash
ollama pull bge-m3           # 약 1.2GB. 컨테이너라면 docker exec -it ollama ollama pull bge-m3

curl -s http://localhost:11434/api/embed -d '{
  "model": "bge-m3",
  "input": ["컨테이너가 자꾸 죽어요", "OOMKilled 발생 시 메모리 limit 조정 방법"]
}' | jq '{model, dims: (.embeddings[0] | length), count: (.embeddings | length)}'
```

```json
{
  "model": "bge-m3",
  "dims": 1024,
  "count": 2
}
```

`input`에 배열을 넣으면 여러 문장을 **한 번에(배치로)** 임베딩합니다. 색인할 때는 항상 배치로 보내세요.

### 7-2. 코사인 유사도 직접 계산

```python
# similarity.py
import math
import requests

OLLAMA = "http://localhost:11434/api/embed"

def embed(texts: list[str]) -> list[list[float]]:
    r = requests.post(OLLAMA, json={"model": "bge-m3", "input": texts}, timeout=120)
    r.raise_for_status()
    return r.json()["embeddings"]

def cosine(a: list[float], b: list[float]) -> float:
    dot = sum(x * y for x, y in zip(a, b))
    return dot / (math.sqrt(sum(x * x for x in a)) * math.sqrt(sum(y * y for y in b)))

query = "컨테이너가 자꾸 죽어요"
docs = [
    "OOMKilled 발생 시 메모리 limit 조정 방법",
    "Pod가 CrashLoopBackOff 상태일 때 로그 확인하기",
    "사내 식당 이번 주 메뉴 안내",
    "How to fix a container that keeps restarting",
]
q, *d = embed([query] + docs)
for text, vec in sorted(zip(docs, d), key=lambda t: -cosine(q, t[1])):
    print(f"{cosine(q, vec):.3f}  {text}")
```

```
0.6xx  Pod가 CrashLoopBackOff 상태일 때 로그 확인하기
0.6xx  How to fix a container that keeps restarting
0.5xx  OOMKilled 발생 시 메모리 limit 조정 방법
0.3xx  사내 식당 이번 주 메뉴 안내
```

(정확한 숫자는 모델 버전에 따라 다릅니다.) 겹치는 단어가 하나도 없는 문서와 **영어 문서**까지 위로 올라오고, 식당 메뉴는 맨 아래로 내려갑니다. 이것이 의미 검색입니다.

> 💡 **대안 — HF TEI**: 임베딩 전용 서버가 필요하면 `docker run -p 8080:80 -v $PWD/tei:/data ghcr.io/huggingface/text-embeddings-inference:cpu-1.9 --model-id BAAI/bge-m3` 로 띄우고 `POST /embed {"inputs": [...]}` 또는 OpenAI 호환 `/v1/embeddings`로 호출합니다. GPU용 이미지는 태그가 GPU 세대별로 다르니 TEI 문서에서 확인하세요.

---

## 8. 실습 — Qdrant

### 8-1. Qdrant 띄우기

```bash
docker run -d --name qdrant \
  -p 6333:6333 -p 6334:6334 \
  -v qdrant-data:/qdrant/storage \
  qdrant/qdrant:v1.20.1

curl -s http://localhost:6333/ | jq     # {"title": "qdrant - vector search engine", "version": "1.20.1", ...}
```

브라우저로 `http://localhost:6333/dashboard`에 들어가면 웹 대시보드가 보입니다.

### 8-2. 컬렉션 생성 (REST)

```bash
curl -s -X PUT http://localhost:6333/collections/wiki \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": { "size": 1024, "distance": "Cosine" },
    "hnsw_config": { "m": 16, "ef_construct": 100 }
  }' | jq

# 자주 필터할 필드에 payload 인덱스
curl -s -X PUT http://localhost:6333/collections/wiki/index \
  -H "Content-Type: application/json" \
  -d '{ "field_name": "category", "field_schema": "keyword" }' | jq '.status'
```

> ⚠️ `size`는 임베딩 모델의 차원과 **정확히 같아야** 합니다. bge-m3는 1024. 모델을 바꾸면 컬렉션을 새로 만들고 전부 다시 임베딩해야 합니다.

### 8-3. 문서 upsert (Python)

```python
# index_qdrant.py
import requests
from qdrant_client import QdrantClient
from qdrant_client.models import PointStruct

def embed(texts: list[str]) -> list[list[float]]:
    r = requests.post("http://localhost:11434/api/embed",
                      json={"model": "bge-m3", "input": texts}, timeout=300)
    r.raise_for_status()
    return r.json()["embeddings"]

DOCS = [
    ("k8s",    "OOMKilled 발생 시 메모리 limit 조정 방법"),
    ("k8s",    "Pod가 CrashLoopBackOff 상태일 때 로그 확인하기"),
    ("k8s",    "HPA로 CPU 사용률 기반 오토스케일링 설정하기"),
    ("docker", "Docker 볼륨으로 데이터베이스 데이터 영속화하기"),
    ("docker", "멀티 스테이지 빌드로 이미지 크기 줄이기"),
    ("llm",    "vLLM의 gpu-memory-utilization 옵션과 KV 캐시"),
    ("llm",    "LiteLLM 가상 키로 팀별 예산 제한하기"),
    ("office", "사내 식당 이번 주 메뉴 안내"),
]

client = QdrantClient(url="http://localhost:6333")
vectors = embed([text for _, text in DOCS])
points = [
    PointStruct(id=i, vector=vec, payload={"category": cat, "text": text})
    for i, ((cat, text), vec) in enumerate(zip(DOCS, vectors), start=1)
]
client.upsert(collection_name="wiki", points=points, wait=True)
print(client.count(collection_name="wiki"))
```

```bash
python index_qdrant.py
# count=8
```

> 💡 **upsert** = 같은 `id`가 있으면 덮어쓰고, 없으면 새로 넣기. 문서가 수정되면 같은 id로 다시 upsert하면 됩니다. 실무에서는 `id`로 문서 경로의 해시나 UUID를 씁니다.

### 8-4. 검색과 필터

```python
# search_qdrant.py
import sys
import requests
from qdrant_client import QdrantClient
from qdrant_client.models import Filter, FieldCondition, MatchValue

def embed_one(text: str) -> list[float]:
    r = requests.post("http://localhost:11434/api/embed",
                      json={"model": "bge-m3", "input": text}, timeout=60)
    r.raise_for_status()
    return r.json()["embeddings"][0]

client = QdrantClient(url="http://localhost:6333")
question = sys.argv[1] if len(sys.argv) > 1 else "컨테이너가 자꾸 죽어요"
qv = embed_one(question)

print("== 전체 검색 ==")
for p in client.query_points("wiki", query=qv, limit=3, with_payload=True).points:
    print(f"{p.score:.3f}  [{p.payload['category']}] {p.payload['text']}")

print("== category=docker 로 필터 ==")
only_docker = Filter(must=[FieldCondition(key="category", match=MatchValue(value="docker"))])
for p in client.query_points("wiki", query=qv, query_filter=only_docker, limit=3).points:
    print(f"{p.score:.3f}  [{p.payload['category']}] {p.payload['text']}")
```

```bash
python search_qdrant.py "GPU 메모리가 모자라요"
```

```
== 전체 검색 ==
0.5xx  [llm] vLLM의 gpu-memory-utilization 옵션과 KV 캐시
0.4xx  [k8s] OOMKilled 발생 시 메모리 limit 조정 방법
0.4xx  [docker] 멀티 스테이지 빌드로 이미지 크기 줄이기
== category=docker 로 필터 ==
0.4xx  [docker] 멀티 스테이지 빌드로 이미지 크기 줄이기
0.3xx  [docker] Docker 볼륨으로 데이터베이스 데이터 영속화하기
```

같은 검색을 REST로 하면 이렇습니다. 앱이 Python이 아니어도 HTTP만 되면 됩니다.

```bash
QV=$(curl -s http://localhost:11434/api/embed -d '{"model":"bge-m3","input":"GPU 메모리가 모자라요"}' | jq -c '.embeddings[0]')

curl -s -X POST http://localhost:6333/collections/wiki/points/query \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": $QV,
    \"filter\": { \"must\": [ { \"key\": \"category\", \"match\": { \"value\": \"k8s\" } } ] },
    \"params\": { \"hnsw_ef\": 128 },
    \"limit\": 2,
    \"with_payload\": true
  }" | jq '.result.points[] | {score, text: .payload.text}'
```

> 💡 필터는 **검색과 동시에** 적용됩니다(먼저 top-k를 뽑고 나서 거르는 것이 아님). 그래서 "부서별 문서만", "2025년 이후 문서만" 같은 조건이 붙어도 결과 개수가 줄지 않습니다. Day 48에서 **권한별 문서 필터링**에 이 기능을 씁니다.

### 8-5. FastAPI 검색 API

Day 42의 RAG 서버가 호출할 검색 엔드포인트를 미리 만들어 둡시다.

```python
# app.py
import requests
from fastapi import FastAPI
from qdrant_client import QdrantClient
from qdrant_client.models import Filter, FieldCondition, MatchValue

app = FastAPI()
qdrant = QdrantClient(url="http://localhost:6333")

def embed_one(text: str) -> list[float]:
    r = requests.post("http://localhost:11434/api/embed",
                      json={"model": "bge-m3", "input": text}, timeout=60)
    r.raise_for_status()
    return r.json()["embeddings"][0]

@app.get("/search")
def search(q: str, k: int = 3, category: str | None = None):
    flt = None
    if category:
        flt = Filter(must=[FieldCondition(key="category", match=MatchValue(value=category))])
    hits = qdrant.query_points("wiki", query=embed_one(q), query_filter=flt, limit=k).points
    return [{"score": round(h.score, 4), **h.payload} for h in hits]
```

```bash
uvicorn app:app --port 9000 &
curl -s "http://localhost:9000/search?q=이미지가%20너무%20커요&k=2" | jq
```

---

## 9. 실습 — pgvector로 같은 일 해 보기

이미 PostgreSQL을 운영 중이라면(Day 06의 DB 컨테이너처럼) **확장 하나만 추가**해 벡터 검색을 할 수 있습니다.

```bash
docker run -d --name pgvector \
  -e POSTGRES_PASSWORD=postgres \
  -p 5433:5432 \
  pgvector/pgvector:0.8.7-pg17

docker exec -it pgvector psql -U postgres -c "CREATE EXTENSION IF NOT EXISTS vector;"
docker exec -it pgvector psql -U postgres -c "
CREATE TABLE wiki (
  id        bigserial PRIMARY KEY,
  category  text,
  body      text,
  embedding vector(1024)
);
CREATE INDEX ON wiki USING hnsw (embedding vector_cosine_ops);"
```

```python
# pg_demo.py
import requests
import psycopg

def embed(texts):
    r = requests.post("http://localhost:11434/api/embed",
                      json={"model": "bge-m3", "input": texts}, timeout=300)
    r.raise_for_status()
    return r.json()["embeddings"]

def to_pg(vec):          # pgvector의 텍스트 표현: '[0.1,0.2,...]'
    return "[" + ",".join(f"{x:.6f}" for x in vec) + "]"

DOCS = [("k8s", "OOMKilled 발생 시 메모리 limit 조정 방법"),
        ("docker", "멀티 스테이지 빌드로 이미지 크기 줄이기"),
        ("llm", "vLLM의 gpu-memory-utilization 옵션과 KV 캐시"),
        ("office", "사내 식당 이번 주 메뉴 안내")]

with psycopg.connect("postgresql://postgres:postgres@localhost:5433/postgres") as conn:
    vecs = embed([t for _, t in DOCS])
    conn.execute("TRUNCATE wiki")
    for (cat, body), v in zip(DOCS, vecs):
        conn.execute("INSERT INTO wiki (category, body, embedding) VALUES (%s, %s, %s::vector)",
                     (cat, body, to_pg(v)))

    q = to_pg(embed(["GPU 메모리가 모자라요"])[0])
    rows = conn.execute("""
        SELECT body, 1 - (embedding <=> %s::vector) AS cosine_similarity
        FROM wiki
        WHERE category <> 'office'              -- 일반 SQL WHERE가 곧 필터
        ORDER BY embedding <=> %s::vector
        LIMIT 3""", (q, q)).fetchall()
    for body, sim in rows:
        print(f"{sim:.3f}  {body}")
```

```bash
python pg_demo.py
```

핵심은 `ORDER BY embedding <=> 질문벡터 LIMIT k` 한 줄입니다. `<=>`는 코사인 **거리**라서 `1 - 거리`가 유사도입니다. 검색 정확도를 올리려면 세션에서 `SET hnsw.ef_search = 100;`처럼 후보 수를 늘립니다.

### Qdrant vs pgvector

| 구분 | Qdrant | pgvector |
|---|---|---|
| 정체 | 벡터 검색 전용 DB (Rust) | PostgreSQL 확장 |
| 도입 비용 | 컨테이너 하나 추가 | 기존 Postgres에 `CREATE EXTENSION` |
| 데이터 모델 | 벡터 + JSON payload | 일반 테이블의 `vector` 컬럼 |
| 필터 | payload 필터 (검색 중 적용) | SQL `WHERE`, JOIN, 트랜잭션 전부 사용 |
| 인덱스 | HNSW (기본), 양자화로 메모리 절약 | HNSW, IVFFlat |
| 규모 | 수억 벡터, 분산(샤딩·복제) 지원 | 수백만 수준까지 무난. 그 이상은 튜닝 필요 |
| 운영 | 별도 백업·모니터링 | 기존 Postgres 운영 체계 그대로 |
| 추천 | 벡터 검색이 핵심 기능, 대규모 | 이미 Postgres가 있고, 메타데이터와 JOIN이 많을 때 |

> 💡 시작은 pgvector, 커지면 Qdrant 같은 전용 DB로 — 도 흔한 경로입니다. 이 시리즈의 RAG(Day 42~43)는 Qdrant로 진행합니다.

### 9-1. 정리

```bash
docker rm -f pgvector
# Qdrant는 Day 42에서 계속 사용하니 남겨 둡니다. 지우려면: docker rm -f qdrant && docker volume rm qdrant-data
```

---

## 10. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| `Wrong input: Vector dimension error: expected dim: 1024, got 384` | 컬렉션 `size`와 임베딩 모델 차원이 다름 | 모델을 확인하고 맞는 `size`로 컬렉션 재생성 |
| `model "bge-m3" not found` | Ollama에 모델이 없음 | `ollama pull bge-m3` |
| 한국어 검색 결과가 엉뚱함 | 영어 전용 임베딩 모델(all-MiniLM 등) 사용 | bge-m3, KURE-v1 등 다국어/한국어 모델로 교체 후 **재색인** |
| e5 모델 점수가 전부 비슷함 | `query: ` / `passage: ` 접두사 누락 | 질문에는 `query: `, 문서에는 `passage: ` 붙이기 |
| 컨테이너 재생성 후 컬렉션이 사라짐 | 스토리지 볼륨 미지정 | `-v qdrant-data:/qdrant/storage` (Day 04) |
| pgvector `type "vector" does not exist` | 확장 미생성 | `CREATE EXTENSION vector;` (DB마다 한 번) |
| pgvector 인덱스를 만들었는데 결과가 적게 나옴 | HNSW 검색 후 `WHERE`로 걸러져 후보 부족 | `hnsw.ef_search` 증가, 또는 0.8+의 `hnsw.iterative_scan` 사용 |

---

## 11. 핵심 정리

- **임베딩** = 텍스트 → 고정 길이 벡터. 의미가 비슷하면 방향이 비슷
- **코사인 유사도** = 두 벡터의 방향 비교(1에 가까울수록 유사). 정규화하면 내적과 같음
- 한국어 문서에는 **다국어/한국어 모델**(bge-m3, KURE-v1, Qwen3-Embedding 등). 컬렉션 차원 = 모델 차원
- 벡터 DB는 **ANN 인덱스(HNSW)**로 수백만 벡터에서도 빠르게 top-k 검색. `m`, `ef`로 정확도↔속도 조절
- Qdrant: Collection / Point(id + vector + payload) / payload 필터. 포트 6333(REST), 6334(gRPC)
- pgvector: `vector(n)` 컬럼 + `<=>` 연산자 + HNSW 인덱스. 기존 Postgres에 바로 추가

---

## 12. 셀프 체크

<details>
<summary>Q1. 키워드 검색으로는 못 찾는데 임베딩 검색으로는 찾을 수 있는 예를 들어 보라.</summary>

"컨테이너가 자꾸 죽어요" → "OOMKilled 발생 시 메모리 limit 조정 방법". 겹치는 단어가 없지만 의미가 가까워 임베딩 벡터의 방향이 비슷합니다. 한국어 질문으로 영어 문서를 찾는 것도 다국어 모델이면 가능합니다.
</details>

<details>
<summary>Q2. 임베딩 모델을 all-MiniLM-L6-v2에서 bge-m3로 바꾸려 한다. 기존 Qdrant 컬렉션은 어떻게 해야 하나?</summary>

차원(384 → 1024)과 벡터 공간 자체가 다르므로 기존 벡터와 섞을 수 없습니다. `size: 1024`로 새 컬렉션을 만들고 **모든 문서를 새 모델로 다시 임베딩**해 넣은 뒤 교체해야 합니다. (차원이 같더라도 모델이 다르면 재색인 필요)
</details>

<details>
<summary>Q3. HNSW에서 검색 시 <code>ef</code> 값을 키우면 어떤 일이 생기나?</summary>

검색할 때 살펴보는 후보 노드가 늘어나 실제 최근접 이웃을 놓칠 확률이 줄어듭니다(recall↑). 대신 비교 횟수가 늘어 지연 시간이 증가합니다.
</details>

<details>
<summary>Q4. Qdrant의 payload 필터가 "검색 후 거르기"가 아니라 "검색 중 적용"이라는 점이 왜 중요한가?</summary>

top-k를 먼저 뽑은 뒤 거르면 조건에 맞는 결과가 0~1개만 남을 수 있습니다. 검색 중에 필터를 적용하면 조건을 만족하는 벡터 중에서 top-k를 찾으므로 요청한 개수만큼 결과를 받을 수 있습니다. 권한·부서별 문서 제한에 필수입니다.
</details>

<details>
<summary>Q5. 이미 Postgres로 서비스를 운영 중인 팀이 RAG를 시작한다. Qdrant와 pgvector 중 무엇으로 시작하겠나?</summary>

문서 수가 수백만 이하이고 메타데이터 JOIN·트랜잭션이 중요하다면 **pgvector**로 시작하는 것이 운영 부담이 적습니다. 벡터 수가 매우 크거나, 벡터 검색 성능·양자화·분산이 핵심 요구라면 **Qdrant** 같은 전용 DB가 유리합니다.
</details>

---

## ⏭️ 다음 예고 — Day 42. RAG 파이프라인 직접 만들기 (청킹·검색·생성)

오늘은 한 줄짜리 문서를 넣었지만, 실제 문서는 수십 페이지입니다. 긴 문서를 어떻게 **청크**로 자를지, 검색한 청크를 어떻게 **프롬프트에 넣을지**, 그리고 Day 40의 게이트웨이를 통해 LLM이 **출처와 함께 답변**하게 만드는 RAG 파이프라인을 처음부터 끝까지 만들어 봅니다.
