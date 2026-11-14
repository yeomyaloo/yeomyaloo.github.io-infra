---
title: "Day 71. 시크릿·인증서 — OpenBao/Vault, cert-manager"
date: 2026-11-15T03:27:54+09:00
weight: 71
tags: ["security", "vault", "cert-manager"]
summary: "인증서를 사람이 손으로 갱신하면 안 되는 이유와 cert-manager의 리소스 구조(Issuer → Certificate → Secret)를 설명할 수 있다."
image: "./images/day71-cert-manager-flow.svg"
draft: false
---
> **Chapter 9 · 보안** | 난이도 ★★★★☆ | 예상 소요 2.5 ~ 3시간

## 🎯 오늘의 목표

- 인증서를 **사람이 손으로 갱신하면 안 되는 이유**와 cert-manager의 리소스 구조(Issuer → Certificate → Secret)를 설명할 수 있다
- 자체 서명 CA로 **ClusterIssuer**를 만들고, Day 15의 Traefik **Gateway HTTPS 리스너**에 인증서를 자동으로 붙일 수 있다
- Let's Encrypt **ACME HTTP-01 / DNS-01** 검증의 차이와 각각을 언제 쓰는지 안다
- **OpenBao**를 dev 모드로 띄워 KV 시크릿, **Kubernetes 인증**, **동적 DB 자격 증명**을 다룰 수 있다
- Day 55의 External Secrets가 **토큰 없이** OpenBao를 SecretStore로 쓰게 만들 수 있다

---

## 1. 오늘 다루는 두 종류의 "비밀"

| 구분 | 인증서 (TLS) | 시크릿 (비밀번호·API 키·DB 자격 증명) |
|---|---|---|
| 성격 | 공개키 + **만료일이 있는** 신원 증명 | 값 자체가 비밀 |
| 대표 사고 | 만료로 인한 전면 장애 | 유출, 하드코딩, 회전 안 됨 |
| 오늘의 도구 | **cert-manager** | **OpenBao** (Vault 포크) + External Secrets |
| 핵심 원칙 | **자동 발급·자동 갱신** | **중앙 보관, 짧은 수명, 접근 감사** |

> 💡 Day 70에서 Istio가 **워크로드 간 mTLS 인증서**를 알아서 발급·회전했습니다. 오늘 cert-manager는 그보다 바깥, **사용자가 접속하는 Ingress/Gateway 인증서**와 클러스터 내부 CA를 담당합니다. Day 13에서 Secret은 base64일 뿐 암호화가 아니라는 것, Day 55에서 SOPS·External Secrets로 "Git에 값을 두지 않는 법"을 봤습니다. 오늘은 그 **값을 어디에 어떻게 보관하고 꺼내는가**를 다룹니다.

---

## 2. cert-manager — 인증서를 쿠버네티스 리소스로

### 2-1. 왜 자동화인가

Day 15에서는 `openssl`로 인증서를 만들고 `kubectl create secret tls`로 넣었습니다. 서비스가 10개, 100개가 되면 문제가 생깁니다.

- 누가 언제 만료되는지 **아무도 모른다** → 어느 날 새벽 브라우저 경고와 함께 장애
- 개인키가 누군가의 노트북 `~/Downloads`에 남는다
- Let's Encrypt 인증서는 **90일** 유효(그리고 더 짧아지는 추세)라 손으로는 감당할 수 없다

cert-manager는 "**이런 인증서가 필요하다**"를 선언하면 발급하고, 만료 전에 **알아서 갱신**해 Secret을 최신으로 유지하는 컨트롤러입니다.

### 2-2. 리소스 구조

![cert-manager 흐름](./images/day71-cert-manager-flow.svg)

| 리소스 | 범위 | 역할 |
|---|---|---|
| **Issuer** / **ClusterIssuer** | 네임스페이스 / 클러스터 | "누가 서명하나" — selfSigned, **CA**, **ACME**(Let's Encrypt), Vault/OpenBao 등 |
| **Certificate** | 네임스페이스 | "이 이름(dnsNames)으로, 이 기간, 이 Secret에" — 원하는 상태 |
| CertificateRequest | 네임스페이스 | Certificate가 만드는 1회성 서명 요청 (CSR 포함) |
| Order / Challenge | 네임스페이스 | ACME 전용 — 도메인 소유 검증 과정 |
| **Secret** (`kubernetes.io/tls`) | 네임스페이스 | 결과물: `tls.crt`, `tls.key`, `ca.crt` |

Certificate를 직접 만들 수도 있고, **Ingress/Gateway에 어노테이션만 붙이면** cert-manager가 Certificate를 대신 만들어 줍니다(ingress-shim / gateway-shim).

> 💡 기본값으로 cert-manager는 유효 기간의 **2/3 지점**에서 갱신합니다(90일 인증서면 약 30일 남았을 때). `renewBefore`로 조정할 수 있습니다.

---

## 3. 실습 ① cert-manager 설치와 자체 CA

**실습 환경**: Day 09의 kind `infra-lab`, Day 15의 Traefik(Helm 차트 42.4.0, Gateway API 활성화). cert-manager **v1.21.3**.

### 3-1. 설치 (Gateway API 지원 켜기)

cert-manager는 Gateway API 지원이 **기본으로 꺼져** 있습니다. 또 시작 시점에 Gateway API CRD가 있는지 확인하므로 CRD가 먼저 있어야 합니다. Day 15에서 이미 설치했으니 그대로 진행합니다.

```yaml
# cert-manager-values.yaml
crds:
  enabled: true          # CRD를 차트와 함께 설치
config:
  gatewayAPI:
    enabled: true        # Gateway 어노테이션 → Certificate 자동 생성
```

```bash
kubectl get crd gateways.gateway.networking.k8s.io     # 있어야 함 (Day 15)

helm install cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --version v1.21.3 \
  --namespace cert-manager --create-namespace \
  -f cert-manager-values.yaml

kubectl get pods -n cert-manager
```

```
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c8b8d6f7d-x7k2p              1/1     Running   0          40s
cert-manager-cainjector-7d9f6b5c4-p2m8q    1/1     Running   0          40s
cert-manager-webhook-6b7c9d8f5-k9n3r       1/1     Running   0          40s
```

| Pod | 역할 |
|---|---|
| cert-manager | 메인 컨트롤러 (Certificate → 발급·갱신) |
| webhook | cert-manager 리소스 검증 어드미션 웹훅 |
| cainjector | 웹훅·APIService 설정에 CA 번들을 주입 |

### 3-2. 자체 서명 CA 만들기 (부트스트랩)

사설 CA는 세 단계로 만듭니다. ① 자기 자신에게 서명하는 Issuer → ② 그걸로 **CA 인증서**(isCA) 발급 → ③ 그 CA로 서명하는 ClusterIssuer.

```yaml
# lab-ca.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-bootstrap
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: lab-root-ca
  namespace: cert-manager          # ClusterIssuer가 읽는 Secret은 이 네임스페이스에 있어야 함
spec:
  isCA: true
  commonName: infra-lab Root CA
  secretName: lab-root-ca
  duration: 87600h                 # 10년 (실습용)
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: selfsigned-bootstrap
    kind: ClusterIssuer
    group: cert-manager.io
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: lab-ca
spec:
  ca:
    secretName: lab-root-ca
```

```bash
kubectl apply -f lab-ca.yaml
kubectl get clusterissuer
```

```
NAME                   READY   AGE
lab-ca                 True    10s
selfsigned-bootstrap   True    10s
```

> ⚠️ ClusterIssuer가 참조하는 Secret은 **cert-manager가 설치된 네임스페이스**(기본 `cert-manager`, "cluster resource namespace")에서 찾습니다. 다른 네임스페이스에 CA Secret을 두면 `secret "lab-root-ca" not found`로 READY가 False가 됩니다.

### 3-3. Certificate 리소스로 직접 발급

```yaml
# cert-whoami.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: whoami-tls
  namespace: tls-demo
spec:
  secretName: whoami-tls
  dnsNames:
  - whoami.example.test
  duration: 2160h       # 90일
  renewBefore: 360h     # 만료 15일 전 갱신
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: lab-ca
    kind: ClusterIssuer
    group: cert-manager.io
```

```bash
kubectl create namespace tls-demo
kubectl apply -f cert-whoami.yaml
kubectl get certificate,certificaterequest -n tls-demo
```

```
NAME                                     READY   SECRET       AGE
certificate.cert-manager.io/whoami-tls   True    whoami-tls   5s

NAME                                              APPROVED   DENIED   READY   ISSUER   REQUESTER                                         AGE
certificaterequest.cert-manager.io/whoami-tls-1   True                True    lab-ca   system:serviceaccount:cert-manager:cert-manager   5s
```

```bash
kubectl get secret whoami-tls -n tls-demo -o jsonpath='{.data.tls\.crt}' | base64 -d \
  | openssl x509 -noout -issuer -enddate -ext subjectAltName
```

```
issuer=CN=infra-lab Root CA
notAfter=Feb 13 09:12:40 2027 GMT
X509v3 Subject Alternative Name:
    DNS:whoami.example.test
```

Day 15에서 openssl 명령 세 줄로 하던 일을 **선언 하나**로 바꿨고, 이제 갱신도 cert-manager의 몫입니다.

### 3-4. Gateway 어노테이션으로 TLS 자동 연결

Day 15의 Traefik 차트는 HTTP 리스너(`web`, 포트 8000)만 가진 `traefik-gateway`를 만들었습니다. 이번에는 `tls-demo` 네임스페이스에 **HTTPS 리스너를 가진 Gateway**를 하나 더 만들고, 어노테이션으로 인증서를 자동 발급받습니다.

> 💡 Traefik은 Gateway 리스너의 `port`가 **Traefik 엔트리포인트 포트와 같아야** 합니다. 차트 기본값에서 `web`은 8000, `websecure`는 **8443**입니다. (Service 포트 443 → 컨테이너 8443)

```yaml
# tls-gateway.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
  namespace: tls-demo
spec:
  selector:
    matchLabels: { app: whoami }
  template:
    metadata:
      labels: { app: whoami }
    spec:
      containers:
      - name: whoami
        image: traefik/whoami:v1.12.0
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: whoami
  namespace: tls-demo
spec:
  selector: { app: whoami }
  ports:
  - port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: tls-gw
  namespace: tls-demo
  annotations:
    cert-manager.io/cluster-issuer: lab-ca      # ← 이 한 줄이 Certificate를 만든다
spec:
  gatewayClassName: traefik
  listeners:
  - name: https
    hostname: gw-tls.example.test               # 필수: dnsNames로 복사됨
    port: 8443                                   # Traefik websecure 엔트리포인트
    protocol: HTTPS
    allowedRoutes:
      namespaces:
        from: Same
    tls:
      mode: Terminate                            # Passthrough는 cert-manager 대상 아님
      certificateRefs:
      - name: gw-tls-cert                        # 이 이름으로 Certificate·Secret 생성
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: whoami
  namespace: tls-demo
spec:
  parentRefs:
  - name: tls-gw
  hostnames:
  - gw-tls.example.test
  rules:
  - backendRefs:
    - name: whoami
      port: 80
```

```bash
kubectl apply -f tls-gateway.yaml
kubectl get certificate -n tls-demo
# NAME          READY   SECRET        AGE
# gw-tls-cert   True    gw-tls-cert   8s      ← 우리가 만들지 않은 Certificate가 생김
# whoami-tls    True    whoami-tls    3m

kubectl get gateway tls-gw -n tls-demo
# NAME     CLASS     ADDRESS   PROGRAMMED   AGE
# tls-gw   traefik             True         10s
```

Day 15와 같은 방법으로 443 포트를 터널링하고, 이번에는 `-k` 없이 **우리 CA를 신뢰**해서 검증합니다.

```bash
# 루트 CA 인증서 꺼내기 (selfSigned라 tls.crt가 곧 CA 인증서)
kubectl get secret lab-root-ca -n cert-manager -o jsonpath='{.data.tls\.crt}' | base64 -d > lab-ca.crt

# 다른 터미널에서
kubectl -n traefik port-forward svc/traefik 8443:443

curl --cacert lab-ca.crt \
  --connect-to gw-tls.example.test:443:127.0.0.1:8443 \
  https://gw-tls.example.test/ | head -3
```

```
Hostname: whoami-6f8d7c9b5-q4r7t
IP: 127.0.0.1
IP: 10.244.1.21
```

`-k` 없이 성공했다는 것은 **인증서 체인 검증과 호스트 이름 검증이 모두 통과**했다는 뜻입니다. 사내 서비스라면 이 `lab-ca.crt`를 사내 PC·서버의 신뢰 저장소에 배포하면 됩니다.

> 💡 Ingress를 쓴다면 같은 방식으로 Ingress에 `cert-manager.io/cluster-issuer: lab-ca` 어노테이션과 `spec.tls`(hosts, secretName)를 적으면 됩니다(ingress-shim, 별도 설정 없이 기본 활성).

---

## 4. Let's Encrypt — ACME HTTP-01과 DNS-01

![ACME HTTP-01 vs DNS-01](./images/day71-acme-challenges.svg)

홈랩 H03에서 Caddy가 Let's Encrypt 인증서를 **자동으로** 받아 오는 걸 봤습니다. 그 뒤에서 돌던 프로토콜이 **ACME**(RFC 8555)입니다. CA는 "이 도메인이 정말 당신 것인가"를 **챌린지**로 확인합니다.

| | **HTTP-01** | **DNS-01** |
|---|---|---|
| 증명 방법 | `http://<도메인>/.well-known/acme-challenge/<토큰>` 에 응답 | `_acme-challenge.<도메인>` **TXT 레코드** 생성 |
| 필요한 것 | 인터넷에서 **80 포트**로 접근 가능 | DNS 제공자 **API 토큰** (Cloudflare, Route53 등) |
| 와일드카드 `*.example.com` | **불가** | **가능** |
| 사설망·CGNAT | 불가 | **가능** (외부 노출 없이 발급) |
| cert-manager 동작 | 임시 Pod + Ingress/HTTPRoute를 만들어 응답 | DNS API로 TXT 생성 → 검증 후 삭제 |
| 홈랩 추천 | 포트포워딩된 서버 | Cloudflare 등으로 도메인을 관리할 때 |

```yaml
# letsencrypt-staging.yaml (개념 예시 — 실제 공개 도메인이 있어야 발급됨)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory   # 먼저 스테이징으로!
    email: you@example.com                    # 만료 알림 등 연락처
    privateKeySecretRef:
      name: letsencrypt-staging-account-key   # ACME 계정 키 (cert-manager가 생성)
    solvers:
    - http01:                                 # ① 기본: HTTP-01, Day 15 Gateway로 응답
        gatewayHTTPRoute:
          parentRefs:
          - name: traefik-gateway
            namespace: traefik
            kind: Gateway
    - selector:                               # ② example.com 존은 DNS-01 (와일드카드 가능)
        dnsZones: ["example.com"]
      dns01:
        cloudflare:
          apiTokenSecretRef:                  # cert-manager 네임스페이스의 Secret
            name: cloudflare-api-token
            key: api-token
```

| 단계 | 리소스 | 확인 명령 |
|---|---|---|
| 1. 발급 요청 | Certificate → CertificateRequest | `kubectl describe certificate <이름>` |
| 2. 주문 | **Order** | `kubectl get order` |
| 3. 검증 | **Challenge** (`pending` → `valid`) | `kubectl describe challenge` — 실패 이유가 여기 |
| 4. 완료 | Secret 갱신 | `kubectl get certificate` READY True |

> ⚠️ Let's Encrypt 운영 서버는 **발급 횟수 제한(rate limit)** 이 있습니다(H03에서 본 그 문제). 설정이 확정될 때까지 **스테이징** ClusterIssuer로 시험하고, 성공하면 `server`만 `https://acme-v02.api.letsencrypt.org/directory`로 바꾼 운영용 ClusterIssuer를 따로 만드세요. 스테이징 인증서는 브라우저가 신뢰하지 않는 것이 정상입니다.

> 💡 Let's Encrypt는 인증서 유효 기간을 90일보다 더 짧게 줄여 가는 계획을 발표했습니다. 수명이 짧아질수록 "사람이 갱신"은 불가능해지고 cert-manager 같은 자동화가 전제가 됩니다.

---

## 5. OpenBao — 시크릿을 금고에

### 5-1. Vault와 OpenBao

| 시점 | 사건 |
|---|---|
| 2023-08 | HashiCorp가 Vault를 포함한 제품 라이선스를 MPL 2.0 → **BSL 1.1**(Business Source License)로 변경. 경쟁 서비스로 제공하는 사용이 제한됨 |
| 2023-12 | 마지막 MPL 버전(1.14 계열)에서 포크한 **OpenBao**가 Linux Foundation 산하에서 출범 (현재는 OpenSSF 프로젝트, MPL 2.0) |
| 2025 | IBM의 HashiCorp 인수 완료 |
| 현재 | OpenBao **v2.7.2**, Helm 차트 0.30.7. CLI는 `bao`, API·개념은 Vault와 거의 같음 |

> 💡 Terraform과 OpenTofu(Day 35)의 관계와 같습니다. 오늘 배우는 개념(시크릿 엔진, 인증 방법, 정책, 리스)은 Vault에도 그대로 적용됩니다. Day 55의 6-3에서는 Vault dev 모드를 root 토큰으로 썼습니다. 오늘은 OpenBao로 **토큰 없는 인증**까지 갑니다.

### 5-2. 핵심 개념

![OpenBao와 External Secrets](./images/day71-openbao-eso-flow.svg)

| 개념 | 설명 | 오늘의 예 |
|---|---|---|
| **시크릿 엔진** | 비밀을 저장하거나 **생성**하는 플러그인, 경로에 마운트 | `secret/`(KV v2), `database/` |
| **인증 방법** | "누구인가"를 확인해 토큰을 발급 | `kubernetes` (ServiceAccount 토큰으로 로그인) |
| **정책** | 경로별 권한 (HCL) | `secret/data/demo/*` read |
| **리스(lease)** | 동적 비밀의 수명. 만료·폐기되면 실제 자격 증명도 삭제 | DB 사용자 1시간 |

**정적 시크릿 vs 동적 시크릿**

| | 정적 (KV) | **동적** (database 엔진 등) |
|---|---|---|
| 값 | 사람이 넣은 값 | 요청할 때마다 **새로 생성** |
| 공유 | 모든 Pod가 같은 비밀번호 | Pod(요청)마다 **다른 계정** |
| 수명 | 누가 바꿀 때까지 | TTL 후 **자동 삭제** |
| 유출 시 | 전체 회전 필요 | 그 리스만 폐기, 피해 범위 작음 |

---

## 6. 실습 ② OpenBao dev 모드

### 6-1. 설치

> ⚠️ **dev 모드**는 메모리 저장, 자동 unseal, TLS 없음, root 토큰 `root`입니다. Pod가 재시작되면 **모든 데이터가 사라지는 학습 전용** 모드입니다. 운영은 standalone/HA(Raft) 모드 + unseal 키 관리 + TLS가 필요합니다.

```bash
helm repo add openbao https://openbao.github.io/openbao-helm
helm repo update
helm install openbao openbao/openbao --version 0.30.7 \
  --namespace openbao --create-namespace \
  --set server.dev.enabled=true \
  --set injector.enabled=false

kubectl get pods -n openbao
# NAME        READY   STATUS    RESTARTS   AGE
# openbao-0   1/1     Running   0          30s
```

매번 `kubectl exec`를 치기 번거로우니 셸 함수를 만듭니다. 컨테이너 안에는 `BAO_ADDR`가 이미 설정되어 있습니다.

```bash
bao() { kubectl exec -i -n openbao openbao-0 -- env BAO_TOKEN=root bao "$@"; }

bao status | grep -E "Sealed|Version|Storage"
```

```
Sealed          false
Version         2.7.2
Storage Type    inmem
```

### 6-2. KV 시크릿

```bash
bao kv put secret/demo/db username=app password='S3cr3t-from-openbao'
bao kv get secret/demo/db
```

```
== Secret Path ==
secret/data/demo/db
...
====== Data ======
Key         Value
---         -----
password    S3cr3t-from-openbao
username    app
```

> 💡 KV **v2**는 버전 관리가 됩니다. CLI 경로는 `secret/demo/db`지만 실제 API·정책 경로는 `secret/data/demo/db`입니다. 정책을 쓸 때 `data/`를 빼먹는 실수가 가장 흔합니다.

### 6-3. 정책과 Kubernetes 인증

```bash
# 1) 읽기 전용 정책
bao policy write demo-read - <<'EOF'
path "secret/data/demo/*" {
  capabilities = ["read"]
}
EOF

# 2) Kubernetes 인증 활성화 — OpenBao가 클러스터 안에 있으므로 API 서버 주소만 주면 됨
bao auth enable kubernetes
bao write auth/kubernetes/config kubernetes_host=https://kubernetes.default.svc:443

# 3) 역할: secrets-demo 네임스페이스의 두 SA에게 demo-read 정책
bao write auth/kubernetes/role/demo \
  bound_service_account_names=app,eso-openbao \
  bound_service_account_namespaces=secrets-demo \
  policies=demo-read \
  ttl=1h
```

| 동작 | 설명 |
|---|---|
| 로그인 요청 | 클라이언트가 자기 **ServiceAccount JWT**를 `auth/kubernetes/login`에 보냄 |
| 검증 | OpenBao가 쿠버네티스 **TokenReview API**로 토큰이 진짜인지, 어떤 SA인지 확인 |
| 발급 | 역할의 바인딩(SA 이름·네임스페이스)과 맞으면 정책이 붙은 OpenBao 토큰 발급 |

OpenBao가 TokenReview를 호출하려면 `system:auth-delegator` 권한이 필요한데, Helm 차트가 기본값(`server.authDelegator.enabled: true`)으로 ClusterRoleBinding을 만들어 둡니다.

직접 로그인해 봅니다.

```bash
kubectl create namespace secrets-demo
kubectl create serviceaccount app -n secrets-demo
kubectl create serviceaccount eso-openbao -n secrets-demo

JWT=$(kubectl create token app -n secrets-demo --duration=10m)
bao write auth/kubernetes/login role=demo jwt="$JWT"
```

```
Key                                       Value
---                                       -----
token                                     (발급된 토큰 값)
token_duration                            1h
token_renewable                           true
token_policies                            ["default" "demo-read"]
token_meta_service_account_name           app
token_meta_service_account_namespace      secrets-demo
...
```

다른 네임스페이스의 SA로 시도하면 `permission denied`가 납니다. **비밀을 꺼내기 위한 첫 번째 비밀(토큰)이 필요 없어졌습니다.** 쿠버네티스가 이미 발급해 둔 SA 토큰이 곧 신원 증명입니다. (Day 69에서 토큰 자동 마운트를 끈 이유와 짝을 이루는 이야기입니다. **필요한 앱만** 토큰을 갖게 하고, 그 토큰으로 필요한 비밀만 꺼냅니다.)

### 6-4. 동적 DB 자격 증명 (짧은 실습)

PostgreSQL을 하나 띄우고, OpenBao가 요청마다 **임시 DB 사용자**를 만들게 합니다.

```bash
kubectl run postgres -n secrets-demo --image=postgres:17.12 \
  --env=POSTGRES_PASSWORD=bootstrap-pass --port=5432 --expose
kubectl wait -n secrets-demo --for=condition=Ready pod/postgres --timeout=90s

bao secrets enable database

bao write database/config/demo-pg \
  plugin_name=postgresql-database-plugin \
  allowed_roles=readonly \
  connection_url='postgresql://{{username}}:{{password}}@postgres.secrets-demo.svc:5432/postgres?sslmode=disable' \
  username=postgres \
  password=bootstrap-pass

# 관리자 비밀번호를 OpenBao만 아는 값으로 교체 (이후 사람은 모름)
bao write -force database/rotate-root/demo-pg

bao write database/roles/readonly \
  db_name=demo-pg \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl=1h \
  max_ttl=24h

bao read database/creds/readonly
```

```
Key                Value
---                -----
lease_id           database/creds/readonly/2f6a614c-4aa2-7b19-24b9-ad944a8d4de6
lease_duration     1h
lease_renewable    true
password           Xb2-kq9TzR7mP4wLs1aE
username           v-root-readonly-q8Zt3kF1mNvB7wXyA2cD-1794729600
```

```bash
# 받은 계정으로 실제 접속되는지 확인 (값은 위 출력으로 바꾸기)
kubectl exec -n secrets-demo postgres -- \
  psql "postgresql://v-root-readonly-...:Xb2-kq9TzR7mP4wLs1aE@localhost/postgres" -c "select current_user;"

# 리스를 폐기하면 DB 사용자도 즉시 삭제됨
bao lease revoke database/creds/readonly/2f6a614c-4aa2-7b19-24b9-ad944a8d4de6
kubectl exec -n secrets-demo postgres -- psql -U postgres -c "\du" | grep v-root || echo "삭제됨"
```

> 💡 `psql -U postgres`가 비밀번호 없이 되는 이유는 공식 postgres 이미지가 **컨테이너 내부 로컬 소켓**에 trust 인증을 쓰기 때문입니다. 네트워크 접속(OpenBao처럼)은 비밀번호가 필요합니다.

앱이 이 자격 증명을 쓰는 방법은 여러 가지입니다.

| 방법 | 형태 | 특징 |
|---|---|---|
| **External Secrets** (+ `VaultDynamicSecret` 제너레이터) | Kubernetes Secret | GitOps와 잘 맞음. 7장에서 정적 KV로 연결 |
| OpenBao **Agent Injector** | 사이드카가 파일로 렌더링, 자동 갱신 | 앱 수정 없이 파일 읽기 |
| OpenBao **CSI provider** | Secrets Store CSI 볼륨 | Secret 오브젝트를 안 만들 수 있음 |
| 앱이 직접 API 호출 | SDK | 리스 갱신까지 앱이 제어 |

---

## 7. 실습 ③ External Secrets가 OpenBao를 SecretStore로

Day 55에서 설치한 ESO(차트 2.15.0, `external-secrets` 네임스페이스)를 그대로 씁니다. ESO는 OpenBao를 **Vault provider**로 접속합니다(ESO 공식 문서의 OpenBao 항목도 Vault provider 사용을 안내).

```yaml
# eso-openbao.yaml
apiVersion: external-secrets.io/v1
kind: SecretStore
metadata:
  name: openbao
  namespace: secrets-demo
spec:
  provider:
    vault:
      server: "http://openbao.openbao.svc:8200"   # dev 모드라 http
      path: "secret"
      version: "v2"
      auth:
        kubernetes:                               # ← Day 55의 tokenSecretRef 대신
          mountPath: "kubernetes"
          role: "demo"
          serviceAccountRef:
            name: eso-openbao                     # ESO가 이 SA의 토큰을 발급받아 로그인
---
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: demo-db
  namespace: secrets-demo
spec:
  refreshInterval: 1m
  secretStoreRef:
    name: openbao
    kind: SecretStore
  target:
    name: demo-db                                 # 만들어질 Kubernetes Secret
  data:
  - secretKey: DB_PASSWORD
    remoteRef:
      key: demo/db
      property: password
  - secretKey: DB_USER
    remoteRef:
      key: demo/db
      property: username
```

```bash
kubectl apply -f eso-openbao.yaml

kubectl get secretstore,externalsecret -n secrets-demo
```

```
NAME                                      AGE   STATUS   CAPABILITIES   READY
secretstore.external-secrets.io/openbao   10s   Valid    ReadWrite      True

NAME                                         STORETYPE     STORE     REFRESH INTERVAL   STATUS         READY
externalsecret.external-secrets.io/demo-db   SecretStore   openbao   1m                 SecretSynced   True
```

```bash
kubectl get secret demo-db -n secrets-demo -o jsonpath='{.data.DB_PASSWORD}' | base64 -d; echo
# S3cr3t-from-openbao

# OpenBao에서 값을 바꾸면 1분 안에 Secret도 바뀜
bao kv put secret/demo/db username=app password='rotated-456'
```

Day 55와 비교하면 차이는 `auth` 블록 하나지만 의미는 큽니다.

| | Day 55 (Vault dev) | 오늘 (OpenBao) |
|---|---|---|
| ESO → 저장소 인증 | root 토큰을 담은 Secret | **ServiceAccount 토큰** (Kubernetes 인증) |
| 저장소 접속용 비밀 | 클러스터에 존재 (유출 시 전부 열람) | **없음** |
| 권한 범위 | root (전부) | 정책 `demo-read` (해당 경로 읽기만) |

> 💡 Vault 1.21 이상을 쓴다면 Kubernetes 인증 역할에 `audience`를 지정하고 ESO의 `serviceAccountRef.audiences`에도 같은 값을 넣어야 한다고 ESO 문서가 안내합니다. 오늘의 OpenBao 역할은 audience를 지정하지 않았습니다.

### 7-1. 정리

```bash
kubectl delete namespace secrets-demo tls-demo
helm uninstall openbao -n openbao && kubectl delete namespace openbao
# cert-manager는 Day 72 이후에도 쓸 수 있으니 남겨 두어도 됩니다. 지우려면:
# kubectl delete clusterissuer lab-ca selfsigned-bootstrap
# helm uninstall cert-manager -n cert-manager && kubectl delete namespace cert-manager
rm -f lab-ca.crt
```

---

## 8. 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| ClusterIssuer `READY False`, `secret ... not found` | CA Secret이 cert-manager 네임스페이스에 없음 | Certificate의 `namespace: cert-manager` 확인 |
| Gateway에 어노테이션을 달았는데 Certificate가 안 생김 | `config.gatewayAPI.enabled` 미설정 / Gateway CRD가 cert-manager보다 늦게 설치됨 / 리스너에 `hostname` 없음 | values 확인, `kubectl rollout restart deploy cert-manager -n cert-manager`, 리스너 조건(hostname·Terminate·certificateRefs) 확인 |
| Gateway `PROGRAMMED False`, 포트 관련 에러 | 리스너 port가 Traefik 엔트리포인트와 불일치 | websecure는 **8443** |
| curl `SSL certificate problem: unable to get local issuer certificate` | 클라이언트가 사설 CA를 모름 | `--cacert lab-ca.crt` 또는 OS 신뢰 저장소에 추가 |
| ACME Challenge가 `pending`에서 멈춤 | HTTP-01: 인터넷에서 80 접근 불가 / DNS-01: 토큰 권한·존 오류 | `kubectl describe challenge`의 Reason 확인. CGNAT면 DNS-01 |
| Let's Encrypt `too many certificates` | 운영 서버 rate limit | 스테이징으로 테스트, 확정 후 운영 전환 |
| `bao` 명령이 `permission denied` (로그인) | 역할의 SA 이름·네임스페이스 바인딩 불일치 | `bao read auth/kubernetes/role/demo`로 확인 |
| 로그인 시 TokenReview 관련 403 | OpenBao SA에 `system:auth-delegator` 없음 | 차트 `server.authDelegator.enabled=true` 확인 |
| ExternalSecret `SecretSyncedError`, `permission denied` | 정책 경로에 `data/` 누락 | KV v2 정책은 `secret/data/<경로>` |
| openbao-0 재시작 후 모든 값 사라짐 | dev 모드는 메모리 저장 | 정상. 운영은 Raft 스토리지 + unseal |

---

## 9. 핵심 정리

- cert-manager: **Issuer/ClusterIssuer(누가 서명) → Certificate(원하는 상태) → Secret(결과)**, 만료 전 자동 갱신
- 사설 CA는 selfSigned → isCA Certificate → CA ClusterIssuer 순으로 부트스트랩. ClusterIssuer의 Secret은 **cert-manager 네임스페이스**
- Gateway에 `cert-manager.io/cluster-issuer` 어노테이션 + 리스너(hostname, Terminate, certificateRefs) → Certificate 자동 생성. Traefik은 리스너 포트 = 엔트리포인트 포트(8443)
- ACME: **HTTP-01**은 80 포트 공개 필요·와일드카드 불가, **DNS-01**은 DNS API 토큰 필요·와일드카드·사설망 가능. 먼저 **스테이징**
- Vault는 2023년 BSL로 전환, **OpenBao**는 MPL 포크. 개념과 API는 거의 같고 CLI는 `bao`
- OpenBao **Kubernetes 인증**: SA 토큰 → TokenReview → 정책 붙은 토큰. "첫 번째 비밀"이 필요 없다
- **동적 시크릿**은 요청마다 새 계정, TTL 후 자동 삭제, 리스 폐기로 즉시 무효화
- ESO는 Vault provider로 OpenBao에 접속. `auth.kubernetes` + `serviceAccountRef`로 토큰 없이 동기화

---

## 10. 셀프 체크

<details>
<summary>Q1. cert-manager에서 Issuer와 ClusterIssuer의 차이, 그리고 ClusterIssuer가 CA Secret을 찾는 위치는?</summary>

Issuer는 같은 네임스페이스의 Certificate만 서명하고, ClusterIssuer는 모든 네임스페이스에서 쓸 수 있습니다. ClusterIssuer가 참조하는 Secret(CA 키, ACME 계정 키, DNS API 토큰 등)은 **cert-manager의 cluster resource namespace(기본 `cert-manager`)** 에서 찾습니다.
</details>

<details>
<summary>Q2. 집 서버가 CGNAT 뒤에 있고 <code>*.home.example.com</code> 와일드카드 인증서가 필요하다. 어떤 챌린지를 써야 하나?</summary>

**DNS-01**. HTTP-01은 인터넷에서 80 포트로 접근할 수 있어야 하고 와일드카드를 발급할 수 없습니다. DNS-01은 DNS 제공자 API로 TXT 레코드만 만들면 되므로 사설망에서도, 와일드카드도 가능합니다.
</details>

<details>
<summary>Q3. Day 55의 tokenSecretRef 방식과 오늘의 Kubernetes 인증 방식의 보안상 차이는?</summary>

tokenSecretRef는 저장소 토큰(그날은 root)이 클러스터 Secret에 들어 있어, 그 Secret이 유출되면 저장소 전체가 열립니다. Kubernetes 인증은 쿠버네티스가 발급한 SA 토큰으로 로그인하고 OpenBao가 TokenReview로 검증하므로 **별도로 보관할 비밀이 없고**, 역할에 묶인 정책 범위(`demo-read`)만 허용됩니다.
</details>

<details>
<summary>Q4. 동적 DB 자격 증명이 정적 비밀번호보다 나은 점 세 가지는?</summary>

① 요청마다 **다른 계정**이라 누가 접근했는지 추적 가능 ② **TTL이 지나면 자동 삭제**되어 오래된 비밀번호가 남지 않음 ③ 유출 시 **해당 리스만 폐기**하면 되므로 전체 회전이 필요 없음. (추가로 rotate-root로 관리자 비밀번호도 사람이 모르게 할 수 있음)
</details>

<details>
<summary>Q5. KV v2에 <code>secret/demo/db</code>로 값을 넣고 정책을 <code>path "secret/demo/*"</code>로 썼더니 permission denied가 난다. 왜인가?</summary>

KV v2의 실제 API 경로는 `secret/data/demo/db`입니다(메타데이터는 `secret/metadata/...`). CLI는 `data/`를 자동으로 붙여 주지만 정책은 실제 경로 기준이므로 `path "secret/data/demo/*"`로 써야 합니다.
</details>

---

## ⏭️ 다음 예고 — Day 72. 런타임 위협 탐지 — Falco

지금까지 Day 69~71에서 **막는 것**(PSA, NetworkPolicy, mTLS, 시크릿 관리)을 쌓았습니다. 하지만 "이미 침입당했다고 가정하라"는 제로 트러스트 원칙대로라면, 막는 것만큼 **뚫렸을 때 알아채는 것**도 중요합니다. 내일은 **Falco**로 컨테이너 안에서 셸이 열리거나, 민감한 파일을 읽거나, 예상치 못한 네트워크 연결이 생기는 순간을 커널 수준에서 탐지하고 알림을 보내 봅니다.
