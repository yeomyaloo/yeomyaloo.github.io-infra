---
title: "CKAD A07. 모의고사 2 — 함정 난이도 (17문항 · 100점)"
date: 2026-11-19T21:39:44+09:00
weight: 8
tags: ["ckad", "certification", "kubernetes", "mock-exam"]
summary: "[공식 커리큘럼 기반 창작 과제 · 실제 시험 문제 아님] 시작 직후 현재 context와 기본 네임스페이스를 의심하는 습관을 만든다. \"kubectl get pods에 아무것도 없다\", \"Service에 엔드포인트가 없다\"처럼 단서가 다른 객체에 있는 상황에서 원인을 찾는다(ReplicaSet 이벤트, probe, --previous 로그)."
image: ""
draft: false
---
> **자격증 준비 · CKAD** | 난이도 ★★★★★ | 예상 소요 시험 2시간 + 채점·복기 2시간

> ⚠️ **이 모의고사의 모든 문항은 CNCF가 공개한 CKAD 커리큘럼(github.com/cncf/curriculum, 2026-11-19 확인 시 `CKAD_Curriculum_v1.37.pdf`)의 5개 도메인을 바탕으로 이 시리즈가 직접 창작한 연습 과제입니다.** 실제 시험 문제나 유출 문제(덤프)가 아니고, 그런 것을 재현하지도 않습니다. Linux Foundation 시험은 응시 정책과 NDA로 보호되므로 "실제 출제 문제"라고 주장하는 자료는 믿지도, 퍼뜨리지도 마세요. 함정의 형태도 실제 시험을 옮긴 것이 아니라, 애플리케이션을 운영하며 실제로 자주 겪는 실수를 모아 만든 것입니다.

> 💡 **실습 환경(A00 실습 환경 고정값 기준).** [A06](./a06-mock-exam-1.md)과 같습니다. `cka-main` = kubeadm v1.37.1(k8s-cp/k8s-w1/k8s-w2, Cilium, 기본 SC `local-path`, Traefik + Gateway API — Gateway `traefik/traefik-gateway`, NodePort `30080`), `cka-aux` = kind. 작업 단말은 base(WSL2), Docker 또는 Podman과 로컬 레지스트리가 있다고 가정하고 **레지스트리 주소는 변수 `REG`** 로 받습니다(A00에서 정한 값). 스크립트와 답안은 `~/ckad/` 아래, 노드 작업은 `ssh <노드>` 후 `sudo -i`, 초기화는 `ckad-clean` 스냅샷입니다.

## 🎯 오늘의 목표

- 시작 직후 **현재 context와 기본 네임스페이스**를 의심하는 습관을 만든다
- "`kubectl get pods`에 아무것도 없다", "Service에 엔드포인트가 없다"처럼 **단서가 다른 객체에 있는** 상황에서 원인을 찾는다(ReplicaSet 이벤트, probe, `--previous` 로그)
- `timeZone`, 네이티브 사이드카, `--reuse-values`, 폐기된 apiVersion처럼 **명령은 성공했는데 결과가 틀리는** 지점을 정확히 짚는다
- 기본 거부 NetworkPolicy 아래에서 **DNS와 Ingress 컨트롤러 경로**까지 열어야 끝나는 연쇄 문항을 푼다

---

## 1. 시험 안내

### 1-1. 형식과 규칙

[A06](./a06-mock-exam-1.md#1-1-형식과-규칙)과 같습니다. 다른 점만 표시합니다.

| 항목 | 이 모의고사 | 비고 |
|---|---|---|
| 제한 시간 | **2시간** | 실제 시험과 같음 |
| 문항 수·배점 | 17문항, 100점, 문항 단위 PASS/FAIL | 도메인 비중은 공식 값(20/20/15/25/20) |
| 합격 참고선 | **66점** | A06보다 낮게 나오는 것이 정상. 첫 시도 50점대면 평균적 |
| 시작 상태 | **current-context가 일부러 바뀌어 있고, 한 context의 기본 네임스페이스도 바뀌어 있음** | 무엇이 바뀌었는지는 풀이 6-1에서 |
| 문항 간 영향 | 한 문항의 조치가 다른 문항의 결과를 막는 경우가 있음 | 시작 5분 동안 문항 전체를 훑을 것 |

### 1-2. 준비와 진행 순서

```bash
# 작업 단말(base)에서. 먼저 A00 방식으로 초기화:
#   vagrant snapshot restore ckad-clean --no-provision   (Vagrantfile 폴더)
#   bash ~/cka/reset-aux.sh                              (cka-aux 재생성)
export REG="<A00에서 정한 레지스트리 host:port>"
mkdir -p ~/ckad && cd ~/ckad
vi a07-setup.sh        # 3절 내용을 붙여 넣고 저장 (LF 줄바꿈)
bash a07-setup.sh      # 이미지 빌드·Helm 설치 포함 4~6분
```

```
[*] preflight ...
[*] files under ~/ckad/a07 ...
[*] building and pushing a07-api:1.0 ...
[*] workloads on cka-main ...
[*] helm release shop ...
[*] objects on cka-aux ...
[*] setting traps ...
[*] ready. Start the 2-hour timer now.
```

> ⚠️ 셋업은 Podman + HTTP 레지스트리라면 push 옵션이 필요할 수 있습니다: `PUSH_OPTS=--tls-verify=false bash a07-setup.sh`. 빌드 도구는 `CTR=podman`처럼 직접 지정할 수도 있습니다.

---

## 2. 문항 한눈에 보기

| # | context | 도메인 | 주제 | 배점 |
|---|---|---|---|---|
| Q1 | cka-main | 설계·빌드 | 고장 난 Dockerfile 고쳐 v2 빌드·push·교체 | 7 |
| Q2 | cka-main | 설계·빌드 | CronJob `timeZone` + 멈춘 CronJob | 7 |
| Q3 | cka-main | 설계·빌드 | 끝나지 않는 Job과 사이드카 | 6 |
| Q4 | cka-main | 배포 | 특정 리비전으로 롤백 | 6 |
| Q5 | cka-main | 배포 | 트래픽이 가지 않는 카나리 | 5 |
| Q6 | cka-main | 배포 | Helm 업그레이드(기존 값 유지) | 5 |
| Q7 | cka-main | 배포 | 빌드되지 않는 Kustomize overlay | 4 |
| Q8 | cka-main | 관측성·유지보수 | 엔드포인트 없는 Service(readiness) | 6 |
| Q9 | cka-main | 관측성·유지보수 | 이전 컨테이너의 로그 | 4 |
| Q10 | cka-main | 관측성·유지보수 | 폐기된 apiVersion 3종 이전 | 5 |
| Q11 | cka-main | 환경·설정·보안 | ResourceQuota 때문에 생기지 않는 Pod | 7 |
| Q12 | cka-main | 환경·설정·보안 | Secret 키·env 파일 ConfigMap | 6 |
| Q13 | cka-main | 환경·설정·보안 | 실행되지 않는 하드닝 Pod + SA | 7 |
| Q14 | **cka-aux** | 환경·설정·보안 | 폐기 예정 버전이 있는 CRD | 5 |
| Q15 | cka-main | 서비스·네트워킹 | 엉뚱한 Pod까지 잡는 Service | 6 |
| Q16 | cka-main | 서비스·네트워킹 | 기본 거부 아래 NetworkPolicy | 7 |
| Q17 | cka-main | 서비스·네트워킹 | 붙지 않는 HTTPRoute | 7 |
| | | | **합계** | **100** |

도메인별 합계: 설계·빌드 20 / 배포 20 / 관측성·유지보수 15 / 환경·설정·보안 25 / 서비스·네트워킹 20.

---

## 3. 사전 준비 스크립트

<details><summary>a07-setup.sh (작업 단말에서 실행 — 시험 후에 열어 보기)</summary>

```bash
#!/bin/bash
# a07-setup.sh -- CKAD mock exam 2 (trap level) setup
# run on the work terminal (base). needs: kubectl, helm, docker|podman, curl
set -euo pipefail

MAIN=${MAIN_CTX:-cka-main}
AUX=${AUX_CTX:-cka-aux}
REG=${REG:?export REG=<registry host:port fixed in A00> first}
REG_SCHEME=${REG_SCHEME:-http}
PUSH_OPTS=${PUSH_OPTS:-}
WORK=~/ckad/a07

if [[ -z "${CTR:-}" ]]; then
  if command -v docker >/dev/null 2>&1; then CTR=docker; else CTR=podman; fi
fi

km() { kubectl --context "$MAIN" "$@"; }
ka() { kubectl --context "$AUX" "$@"; }
ensure_ns() { "$1" create namespace "$2" --dry-run=client -o yaml | "$1" apply -f - >/dev/null; }
die() { echo "ERROR: $*" >&2; exit 1; }

echo "[*] preflight ..."
for c in "$MAIN" "$AUX"; do
  kubectl config get-contexts "$c" >/dev/null 2>&1 || die "context $c not found"
done
km get nodes >/dev/null
ka get nodes >/dev/null
command -v helm >/dev/null 2>&1 || die "helm not found"
command -v "$CTR" >/dev/null 2>&1 || die "$CTR not found (set CTR=docker|podman)"
curl -fsS -m 5 "${REG_SCHEME}://${REG}/v2/" >/dev/null || die "registry ${REG_SCHEME}://${REG} not reachable"
km get gatewayclass traefik >/dev/null 2>&1 || die "GatewayClass traefik not found (CKA K04 Traefik setup)"
km -n traefik get gateway traefik-gateway >/dev/null 2>&1 || die "Gateway traefik/traefik-gateway not found"
km get ingressclass traefik >/dev/null 2>&1 || die "IngressClass traefik not found"
# start from a clean kubeconfig state
kubectl config set-context "$MAIN" --namespace=default >/dev/null
kubectl config use-context "$MAIN" >/dev/null

echo "[*] files under ~/ckad/a07 ..."
rm -rf "$WORK"
mkdir -p "$WORK"/api/secrets "$WORK"/legacy "$WORK"/charts/shop/templates \
         "$WORK"/kustomize/base "$WORK"/kustomize/overlays/prod

# Q1: application source with two bugs (Dockerfile CMD, .dockerignore)
cat > "$WORK/api/server.py" <<'EOF'
from http.server import BaseHTTPRequestHandler, HTTPServer

VERSION = "a07-api v2"


class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        body = (VERSION + "\n").encode()
        self.send_response(200)
        self.send_header("Content-Type", "text/plain")
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)


HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()
EOF
cat > "$WORK/api/Dockerfile" <<'EOF'
FROM python:3.12-alpine
WORKDIR /app
COPY . .
EXPOSE 8000
USER 10001
CMD ["python", "serve.py"]
EOF
cat > "$WORK/api/.dockerignore" <<'EOF'
.git
*.pyc
secrets/
*.py
!main.py
EOF
echo "upstream-token-do-not-ship" > "$WORK/api/secrets/token.txt"

# Q10: manifests using removed APIs
cat > "$WORK/legacy/pdb.yaml" <<'EOF'
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: catalog-pdb
  namespace: a07-obs
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: catalog
EOF
cat > "$WORK/legacy/hpa.yaml" <<'EOF'
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: catalog-hpa
  namespace: a07-obs
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalog
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 70
EOF
cat > "$WORK/legacy/ingress.yaml" <<'EOF'
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: catalog-ing
  namespace: a07-obs
spec:
  ingressClassName: traefik
  rules:
  - host: catalog.a07.test
    http:
      paths:
      - path: /
        backend:
          serviceName: catalog
          servicePort: 80
EOF

# Q12: env file
cat > "$WORK/orders.env" <<'EOF'
# orders worker settings
MODE=batch
REGION=ap-northeast-2
RETRY=3
EOF

# Q13: a pod manifest that cannot run as written
cat > "$WORK/hardened.yaml" <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: hardened
  namespace: a07-sec
spec:
  serviceAccountName: app-runner
  securityContext:
    runAsNonRoot: true
  containers:
  - name: app
    image: busybox:1.37
    command: ["sh", "-c", "touch /tmp/ready && sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
EOF

# Q6: local helm chart
cat > "$WORK/charts/shop/Chart.yaml" <<'EOF'
apiVersion: v2
name: shop
description: small nginx chart for the CKAD mock exam
type: application
version: 0.1.0
appVersion: "1.27"
EOF
cat > "$WORK/charts/shop/values.yaml" <<'EOF'
replicaCount: 1
image:
  repository: nginx
  tag: 1.27-alpine
service:
  type: ClusterIP
  port: 80
EOF
cat > "$WORK/charts/shop/templates/deployment.yaml" <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: web
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
EOF
cat > "$WORK/charts/shop/templates/service.yaml" <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
EOF

# Q7: kustomize overlay whose patch targets a name that does not exist
cat > "$WORK/kustomize/base/deployment.yaml" <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: nginx:1.27-alpine
EOF
cat > "$WORK/kustomize/base/kustomization.yaml" <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
EOF
cat > "$WORK/kustomize/overlays/prod/replicas-patch.yaml" <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 4
EOF
cat > "$WORK/kustomize/overlays/prod/kustomization.yaml" <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: a07-prod
resources:
- ../../base
patches:
- path: replicas-patch.yaml
EOF

echo "[*] building and pushing a07-api:1.0 ..."
tmp=$(mktemp -d)
sed 's/a07-api v2/a07-api v1/' "$WORK/api/server.py" > "$tmp/server.py"
printf 'FROM python:3.12-alpine\nWORKDIR /app\nCOPY server.py .\nEXPOSE 8000\nUSER 10001\nCMD ["python", "server.py"]\n' > "$tmp/Dockerfile"
"$CTR" build -q -t "$REG/ckad/a07-api:1.0" "$tmp" >/dev/null
# shellcheck disable=SC2086
"$CTR" push $PUSH_OPTS "$REG/ckad/a07-api:1.0" >/dev/null
rm -rf "$tmp"

echo "[*] workloads on cka-main ..."
km delete namespace a07-prod --ignore-not-found >/dev/null
for ns in a07-build a07-batch a07-deploy a07-obs a07-quota a07-cfg a07-sec a07-net a07-decoy; do
  ensure_ns km "$ns"
done
km -n default create serviceaccount app-runner --dry-run=client -o yaml | km apply -f - >/dev/null

# Q1 deployment (needs $REG, so unquoted heredoc)
km apply -f - >/dev/null <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: a07-build
spec:
  replicas: 2
  selector:
    matchLabels: {app: api}
  template:
    metadata:
      labels: {app: api}
    spec:
      containers:
      - name: app
        image: ${REG}/ckad/a07-api:1.0
        ports:
        - containerPort: 8000
        readinessProbe:
          httpGet: {path: /, port: 8000}
          periodSeconds: 5
      - name: metrics
        image: busybox:1.37
        command: ["sh", "-c", "sleep 604800"]
EOF

km apply -f - >/dev/null <<'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
  namespace: a07-batch
spec:
  schedule: "*/5 * * * *"
  suspend: true
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: cleanup
            image: busybox:1.37
            command: ["sh", "-c", "echo cleanup done"]
---
apiVersion: batch/v1
kind: Job
metadata:
  name: etl
  namespace: a07-batch
spec:
  backoffLimit: 0
  template:
    spec:
      restartPolicy: Never
      volumes:
      - name: data
        emptyDir: {}
      containers:
      - name: etl
        image: busybox:1.37
        command:
        - sh
        - -c
        - |
          for i in 1 2 3 4 5; do echo "row $i loaded" >> /data/out.log; sleep 2; done
          echo "etl done" >> /data/out.log
        volumeMounts:
        - {name: data, mountPath: /data}
      - name: shipper
        image: busybox:1.37
        command: ["sh", "-c", "touch /data/out.log; tail -F /data/out.log"]
        volumeMounts:
        - {name: data, mountPath: /data}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-stable
  namespace: a07-deploy
spec:
  replicas: 3
  selector:
    matchLabels: {app: web, track: stable}
  template:
    metadata:
      labels: {app: web, track: stable}
    spec:
      containers:
      - name: web
        image: nginx:1.27-alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: a07-deploy
spec:
  selector: {app: web, track: stable}
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalog
  namespace: a07-obs
spec:
  replicas: 2
  selector:
    matchLabels: {app: catalog}
  template:
    metadata:
      labels: {app: catalog}
    spec:
      containers:
      - name: web
        image: nginx:1.27-alpine
        ports:
        - name: http
          containerPort: 80
        readinessProbe:
          httpGet: {path: /, port: 8080}
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: catalog
  namespace: a07-obs
spec:
  selector: {app: catalog}
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: batch-runner
  namespace: a07-obs
spec:
  volumes:
  - name: state
    emptyDir: {}
  containers:
  - name: runner
    image: busybox:1.37
    volumeMounts:
    - {name: state, mountPath: /state}
    command:
    - sh
    - -c
    - |
      if [ ! -f /state/once ]; then
        touch /state/once
        echo "runner 1.4.0 starting"
        echo "FATAL code=E-4417 token rejected by upstream"
        exit 1
      fi
      echo "runner 1.4.0 starting"
      echo "recovered from previous failure, idle"
      sleep 604800
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalog
  namespace: a07-decoy
spec:
  replicas: 2
  selector:
    matchLabels: {app: catalog}
  template:
    metadata:
      labels: {app: catalog}
    spec:
      containers:
      - name: web
        image: nginx:1.27-alpine
        readinessProbe:
          httpGet: {path: /, port: 8080}
---
apiVersion: v1
kind: Service
metadata:
  name: catalog
  namespace: a07-decoy
spec:
  selector: {app: catalog}
  ports:
  - port: 80
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: a07-quota
spec:
  hard:
    pods: "5"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: filler
  namespace: a07-quota
spec:
  replicas: 1
  selector:
    matchLabels: {app: filler}
  template:
    metadata:
      labels: {app: filler}
    spec:
      containers:
      - name: filler
        image: busybox:1.37
        command: ["sh", "-c", "sleep 604800"]
        resources:
          requests: {cpu: 500m, memory: 256Mi}
          limits: {cpu: "1", memory: 512Mi}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: a07-quota
spec:
  replicas: 3
  selector:
    matchLabels: {app: api}
  template:
    metadata:
      labels: {app: api}
    spec:
      containers:
      - name: api
        image: busybox:1.37
        command: ["sh", "-c", "sleep 604800"]
---
apiVersion: v1
kind: Secret
metadata:
  name: orders-db
  namespace: a07-cfg
type: Opaque
stringData:
  DB_PASSWORD: Or4ers-a07
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders
  namespace: a07-cfg
spec:
  replicas: 1
  selector:
    matchLabels: {app: orders}
  template:
    metadata:
      labels: {app: orders}
    spec:
      containers:
      - name: orders
        image: busybox:1.37
        command: ["sh", "-c", "sleep 604800"]
        envFrom:
        - configMapRef:
            name: orders-cfg
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: orders-db
              key: db-password
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pay
  namespace: a07-net
spec:
  replicas: 3
  selector:
    matchLabels: {app: pay, tier: api}
  template:
    metadata:
      labels: {app: pay, tier: api}
    spec:
      containers:
      - name: web
        image: nginx:1.27-alpine
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: pay-debug
  namespace: a07-net
  labels: {app: pay, tier: debug}
spec:
  containers:
  - name: debug
    image: busybox:1.37
    command: ["sh", "-c", "sleep 604800"]
---
apiVersion: v1
kind: Service
metadata:
  name: pay
  namespace: a07-net
spec:
  selector: {app: pay}
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: a07-net
spec:
  replicas: 2
  selector:
    matchLabels: {app: backend}
  template:
    metadata:
      labels: {app: backend}
    spec:
      containers:
      - name: web
        image: nginx:1.27-alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: a07-net
spec:
  selector: {app: backend}
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: a07-net
  labels: {app: frontend}
spec:
  containers:
  - name: cli
    image: busybox:1.37
    command: ["sh", "-c", "sleep 604800"]
---
apiVersion: v1
kind: Pod
metadata:
  name: intruder
  namespace: a07-net
  labels: {app: intruder}
spec:
  containers:
  - name: cli
    image: busybox:1.37
    command: ["sh", "-c", "sleep 604800"]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shop
  namespace: a07-net
spec:
  replicas: 2
  selector:
    matchLabels: {app: shop}
  template:
    metadata:
      labels: {app: shop}
    spec:
      containers:
      - name: web
        image: nginx:1.27-alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: shop
  namespace: a07-net
spec:
  selector: {app: shop}
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: a07-net
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: shop-route
  namespace: a07-net
spec:
  parentRefs:
  - name: traefik-gateway
  hostnames:
  - shop.a07.test
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: shop
      port: 8080
EOF
km -n a07-deploy get svc web -o jsonpath='{.metadata.uid}' > "$WORK/.svc-web-uid"

# Q4: three revisions, then paused
pay_rev() {   # pay_rev <image> <change-cause>
  km apply -f - >/dev/null <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pay
  namespace: a07-deploy
  annotations:
    kubernetes.io/change-cause: "$2"
spec:
  replicas: 3
  selector:
    matchLabels: {app: pay}
  template:
    metadata:
      labels: {app: pay}
    spec:
      containers:
      - name: web
        image: $1
EOF
}
pay_rev nginx:1.27-alpine "known-good"
km -n a07-deploy rollout status deployment/pay --timeout=180s >/dev/null
pay_rev busybox:1.37 "switch base image"
sleep 5
pay_rev nginx:0.0.0-a07 "hotfix 0.0.0"
sleep 3
km -n a07-deploy rollout pause deployment/pay >/dev/null

echo "[*] helm release shop ..."
helm --kube-context "$MAIN" upgrade --install shop "$WORK/charts/shop" -n a07-helm --create-namespace \
  --set service.port=8080 --wait --timeout 3m >/dev/null

echo "[*] objects on cka-aux ..."
for ns in a07-crd a07-batch; do ensure_ns ka "$ns"; done
ka apply -f - >/dev/null <<'EOF'
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backends.platform.a07.example.com
spec:
  group: platform.a07.example.com
  scope: Namespaced
  names:
    plural: backends
    singular: backend
    kind: Backend
    shortNames: [be]
  versions:
  - name: v1beta1
    served: true
    storage: false
    deprecated: true
    deprecationWarning: "platform.a07.example.com/v1beta1 Backend is deprecated; use platform.a07.example.com/v1"
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: [tier, replicas]
            properties:
              tier:
                type: string
                enum: [bronze, silver, gold]
              replicas:
                type: integer
                minimum: 1
                maximum: 5
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: [tier, replicas]
            properties:
              tier:
                type: string
                enum: [bronze, silver, gold]
              replicas:
                type: integer
                minimum: 1
                maximum: 5
EOF

echo "[*] setting traps ..."
kubectl config set-context "$MAIN" --namespace=a07-decoy >/dev/null
kubectl config use-context "$AUX" >/dev/null
echo "[*] ready. Start the 2-hour timer now."
```

</details>

> 💡 CRD의 두 버전(`v1beta1`, `v1`)은 스키마가 같으므로 변환 전략은 기본값(`None`, apiVersion만 바꿔 줌)으로 충분합니다.

---

## 4. 문항

> 공통 조건: 이름·네임스페이스·숫자는 **문항에 적힌 그대로** 써야 채점됩니다. 파일 답안은 작업 단말의 `~/ckad/a07/` 아래. `$REG`는 레지스트리 주소입니다.

### Q1. 이미지 v2 — 7점

```bash
kubectl config use-context cka-main
```

`~/ckad/a07/api`에 Python 앱의 v2 소스가 있다. 이 디렉터리로 `$REG/ckad/a07-api:2.0`을 빌드·push하고, `a07-build`의 Deployment `api`가 이 이미지를 쓰게 하라. 2개 Pod가 모두 Ready이고 응답 본문이 `a07-api v2`여야 한다.

- 빌드한 이미지가 **실제로 동작**해야 한다(필요하면 Dockerfile과 `.dockerignore`를 고친다)
- `.dockerignore`의 `secrets/` 규칙은 유지한다. **`secrets/` 아래 파일이 이미지에 들어가면 안 된다**
- Deployment의 다른 컨테이너(`metrics`)는 바꾸지 않는다

### Q2. CronJob — 7점

```bash
kubectl config use-context cka-main
```

`a07-batch`에서:

1. CronJob `kr-report`를 만들어라. **서울 시각(Asia/Seoul)으로 매일 02:30**에 실행되며, 컨트롤 플레인의 시간대 설정에 의존하지 않아야 한다. 이미지 `busybox:1.37`, 명령 `date`. 이전 실행이 아직 돌고 있으면 새 실행은 **건너뛰고**, 예정 시각에서 **300초**가 지나도록 시작하지 못한 실행은 포기한다.
2. 같은 네임스페이스의 CronJob `cleanup`이 일정대로 한 번도 실행되지 않는다. 원인을 찾아 다음 일정부터 실행되게 하라(스케줄은 바꾸지 않는다).

### Q3. 끝나지 않는 Job — 6점

```bash
kubectl config use-context cka-main
```

`a07-batch`의 Job `etl`이 작업을 마쳤는데도 완료(`Complete`)되지 않는다. 로그 수집 컨테이너 `shipper`가 `etl`과 **함께 동작하면서도** Job이 완료되도록 Job `etl`을 고쳐라. 컨테이너 이름·이미지·명령은 그대로 두고, 최종적으로 Job `etl`이 성공 1이어야 하며 `shipper`의 로그에 `etl done`이 보여야 한다.

### Q4. 롤백 — 6점

```bash
kubectl config use-context cka-main
```

`a07-deploy`의 Deployment `pay`는 최근 두 번의 변경이 모두 문제였다. 변경 사유(CHANGE-CAUSE)가 **`known-good`** 인 리비전의 상태로 되돌려, 3개 Pod가 모두 Ready이고 이후 변경이 정상적으로 롤아웃될 수 있는 상태로 만들어라.

### Q5. 카나리 — 5점

```bash
kubectl config use-context cka-main
```

`a07-deploy`에 Deployment `web-stable`(3개)과 Service `web`이 있다. 카나리 Deployment `web-canary`(이미지 `nginx:1.28-alpine`, 레이블 `app=web,track=canary`, 1개)를 만들고 `web-stable`을 **4개**로 늘려, Service `web`으로 들어오는 요청의 **약 20%** 가 카나리로 가게 하라. Service `web`은 **지우고 다시 만들지 말 것**(이름·포트 유지).

### Q6. Helm 업그레이드 — 5점

```bash
kubectl config use-context cka-main
```

네임스페이스 `a07-helm`에 Helm 릴리스 `shop`이 설치되어 있다(차트 `~/ckad/a07/charts/shop`). 이 릴리스를 업그레이드해 `replicaCount=3`, `image.tag=1.28-alpine`이 되게 하라. **설치 때 지정했던 다른 값은 모두 그대로 유지**해야 한다.

### Q7. Kustomize — 4점

```bash
kubectl config use-context cka-main
```

`~/ckad/a07/kustomize/overlays/prod`는 `kubectl kustomize`로 빌드조차 되지 않는다. **base는 수정하지 말고** overlay 디렉터리 안의 파일만 고쳐 빌드되게 하고, 적용해서 네임스페이스 `a07-prod`에 Deployment `api`가 **4개** Ready가 되게 하라.

### Q8. 엔드포인트 없는 Service — 6점

```bash
kubectl config use-context cka-main
```

`a07-obs`의 Service `catalog`로 요청하면 연결이 안 된다. **Service는 수정하지 말고** 원인을 고쳐 Deployment `catalog`의 Pod 2개가 모두 엔드포인트로 잡히게 하라. readiness probe는 없애지 않는다(HTTP GET `/` 유지).

### Q9. 이전 컨테이너의 로그 — 4점

```bash
kubectl config use-context cka-main
```

`a07-obs`의 Pod `batch-runner`는 지금은 Running이지만 한 번 실패한 적이 있다. **실패했던 컨테이너가 남긴 `FATAL`로 시작하는 줄**을 그대로 `~/ckad/a07/crash.txt`에 저장하라(한 줄).

### Q10. 폐기된 API — 5점

```bash
kubectl config use-context cka-main
```

`~/ckad/a07/legacy/`의 `pdb.yaml`, `hpa.yaml`, `ingress.yaml`은 현재 클러스터(v1.37)에서 적용되지 않는다. **각 파일을 그 자리에서** 현재 제공되는 안정 API로 고치고 모두 적용하라. 동작(최소 가용 1, CPU 사용률 70% 목표, `catalog.a07.test`의 `/` 이하 → Service `catalog:80`)은 그대로 유지한다. Ingress의 경로 매칭은 `Prefix`로 한다.

### Q11. 생기지 않는 Pod — 7점

```bash
kubectl config use-context cka-main
```

`a07-quota`의 Deployment `api`(replicas 3)는 만들어졌지만 Pod가 하나도 없다. 원인을 찾아 **3개 모두 Running**이 되게 하라. 조건:

- ResourceQuota `team-quota`와 Deployment `filler`는 수정·삭제하지 않는다
- `api` 컨테이너 리소스는 requests `memory: 128Mi`, limits `cpu: 300m`, `memory: 256Mi`
- requests `cpu`는 **3개가 모두 쿼터 안에 들어가는 가장 큰 값을 50m 단위로** 정한다

### Q12. 설정이 없는 Deployment — 6점

```bash
kubectl config use-context cka-main
```

`a07-cfg`의 Deployment `orders`의 Pod가 시작하지 못한다. **Deployment는 수정하지 말고** 다음을 고쳐 Pod를 Running으로 만들어라.

- Secret `orders-db`에 Deployment가 기대하는 키가 있어야 한다. 값은 Secret에 이미 들어 있는 비밀번호를 그대로 쓴다
- ConfigMap `orders-cfg`를 `~/ckad/a07/orders.env`로 만들어, 파일의 **각 설정이 별도의 환경 변수**(`MODE`, `REGION`, `RETRY`)로 들어가게 한다

### Q13. 하드닝 Pod — 7점

```bash
kubectl config use-context cka-main
```

`~/ckad/a07/hardened.yaml`의 Pod `hardened`(네임스페이스 `a07-sec`)는 그대로 적용하면 실행되지 않는다. 파일을 고쳐 적용하고 Pod를 Running으로 만들어라. 최종 Pod는:

- ServiceAccount `app-runner`(네임스페이스 `a07-sec`)로 실행되고, API 토큰은 자동 마운트하지 않는다
- `runAsNonRoot: true`를 유지하고 UID `1000`으로 실행된다
- 권한 상승 금지, 루트 파일시스템 읽기 전용, 모든 capability 제거를 유지한다
- 컨테이너 명령은 바꾸지 않는다(`/tmp/ready` 파일이 실제로 만들어져야 함)

### Q14. CRD 버전 — 5점

```bash
kubectl config use-context cka-aux
```

이 클러스터의 `platform.a07.example.com` 그룹 CRD로 네임스페이스 `a07-crd`에 `Backend` 리소스 `orders-backend`를 만들어라(등급 `gold`, 복제 수 `2`). 이 CRD에는 **폐기 예정인 버전**이 있으니, 그 버전으로 만들지 말 것.

### Q15. 엉뚱한 Pod를 잡는 Service — 6점

```bash
kubectl config use-context cka-main
```

`a07-net`의 Service `pay`가 가끔만 응답한다. **Pod와 Deployment는 수정하지 말고** Service만 고쳐, Deployment `pay`의 Pod 3개**만** 엔드포인트가 되고 각 Pod의 `http` 포트로 연결되게 하라.

### Q16. 기본 거부 아래의 통신 — 7점

```bash
kubectl config use-context cka-main
```

`a07-net`에는 모든 인그레스·이그레스를 막는 NetworkPolicy `deny-all`이 있다(수정·삭제 금지). 다음 두 정책을 추가해 Pod `frontend`가 **`http://backend`(Service 이름)** 로 접속할 수 있게 하라.

- `frontend-egress`: `app=frontend` Pod의 이그레스 — `app=backend` Pod의 TCP 80, 그리고 이름 확인에 필요한 최소한의 통신
- `backend-ingress`: `app=backend` Pod의 인그레스 — `app=frontend` Pod에서 오는 TCP 80만

### Q17. 붙지 않는 HTTPRoute — 7점

```bash
kubectl config use-context cka-main
```

`a07-net`의 HTTPRoute `shop-route`를 만들었지만 작업 단말에서 `curl -H 'Host: shop.a07.test' http://192.168.56.11:30080/`이 nginx 페이지를 돌려주지 않는다. 원인을 모두 찾아 고쳐라. Gateway(`traefik/traefik-gateway`)와 `deny-all`은 수정하지 않으며, 필요한 허용은 **새 NetworkPolicy를 추가**해서 해결한다.

---

## 5. 채점

### 5-1. 채점 절차

```bash
cd ~/ckad
vi grade.sh      # 5-3 전문 붙여 넣기
bash grade.sh    # 셋업과 같은 REG 필요
```

```
== CKAD mock exam 2 (a07) grading  2026-11-19 17:41 ==
Q01  FAIL    0/  7  api v2 image (no secrets)
Q02  PASS    7/  7  CronJob timeZone + resume
...
---------------------------------------------------
TOTAL: 58 / 100   (PASS 10, FAIL 7)
RESULT: below reference line (66)
```

### 5-2. 채점 체크리스트

| # | 검증 명령(context 생략 시 cka-main, `-n` 생략 금지) | 기대 결과 |
|---|---|---|
| Q1 | `kubectl -n a07-build exec deploy/api -c app -- wget -qO- localhost:8000` | `a07-api v2`, `/app/secrets` 없음 |
| Q2 | `kubectl -n a07-batch get cronjob` | `kr-report` `30 2 * * *` TIMEZONE `Asia/Seoul`, `cleanup` SUSPEND `False` |
| Q3 | `kubectl -n a07-batch get job etl` | `Complete` 1/1 |
| Q4 | `kubectl -n a07-deploy get deploy pay -o wide` | `3/3`, `nginx:1.27-alpine`, 일시 정지 아님 |
| Q5 | `kubectl -n a07-deploy get endpointslices -l kubernetes.io/service-name=web` | 주소 5개 |
| Q6 | `helm -n a07-helm get values shop` | `service.port: 8080`, `replicaCount: 3`, `image.tag` |
| Q7 | `kubectl -n a07-prod get deploy api` | `4/4` |
| Q8 | `kubectl -n a07-obs get endpointslices -l kubernetes.io/service-name=catalog` | 주소 2개 |
| Q9 | `cat ~/ckad/a07/crash.txt` | `FATAL code=E-4417 ...` |
| Q10 | `kubectl -n a07-obs get pdb,hpa,ingress` | 3개 모두 |
| Q11 | `kubectl -n a07-quota describe quota team-quota` | pods 4/5, requests.cpu 950m/1 |
| Q12 | `kubectl -n a07-cfg exec deploy/orders -- env \| sort` | `MODE`, `REGION`, `RETRY`, `DB_PASSWORD` |
| Q13 | `kubectl -n a07-sec exec hardened -- sh -c 'id; ls /tmp'` | `uid=1000`, `ready` |
| Q14 | `kubectl --context cka-aux -n a07-crd get be orders-backend -o yaml \| grep apiVersion` | managedFields 모두 `.../v1` |
| Q15 | `kubectl -n a07-net get endpointslices -l kubernetes.io/service-name=pay -o wide` | `pay` Pod IP 3개만, 포트 80 |
| Q16 | `kubectl -n a07-net exec frontend -- wget -qO- -T 3 http://backend` | nginx 페이지 |
| Q17 | `curl -s -H 'Host: shop.a07.test' http://192.168.56.11:30080/ \| grep title` | `Welcome to nginx!` |

### 5-3. 자동 채점 스크립트 `grade.sh` 전문

```bash
#!/bin/bash
# grade.sh -- CKAD mock exam 2 (a07) auto grader
# run on the work terminal, like a07-setup.sh. does not change cluster objects.
MAIN=${MAIN_CTX:-cka-main}
AUX=${AUX_CTX:-cka-aux}
REG=${REG:?export REG=<registry host:port fixed in A00> first}
REG_SCHEME=${REG_SCHEME:-http}
NODE_IP=${NODE_IP:-192.168.56.11}
W=~/ckad/a07
SCORE=0; MAX=0; NPASS=0; NFAIL=0

km() { kubectl --context "$MAIN" --request-timeout=20s "$@"; }
ka() { kubectl --context "$AUX" --request-timeout=20s "$@"; }
check() {   # check <no> <points> <title> <function>
  local no=$1 pts=$2 title=$3 fn=$4
  MAX=$((MAX + pts))
  if ( "$fn" ) >/dev/null 2>&1; then
    SCORE=$((SCORE + pts)); NPASS=$((NPASS + 1))
    printf 'Q%02d  PASS  %3d/%3d  %s\n' "$no" "$pts" "$pts" "$title"
  else
    NFAIL=$((NFAIL + 1))
    printf 'Q%02d  FAIL  %3d/%3d  %s\n' "$no" 0 "$pts" "$title"
  fi
}
jp() { "$1" "${@:2:$#-2}" -o jsonpath="${!#}"; }
eq() { [[ "$1" == "$2" ]]; }
ready_eps() {
  jp "$1" -n "$2" get endpointslices -l kubernetes.io/service-name="$3" \
     '{range .items[*].endpoints[*]}{.conditions.ready}{"\n"}{end}' | grep -c true
}
pod_ready() { eq "$(jp "$1" -n "$2" get pod "$3" '{.status.conditions[?(@.type=="Ready")].status}')" True; }

# --- checks ------------------------------------------------------------
q01() {
  eq "$(jp km -n a07-build get deploy api '{.spec.template.spec.containers[?(@.name=="app")].image}|{.spec.template.spec.containers[?(@.name=="metrics")].image}')" \
     "${REG}/ckad/a07-api:2.0|busybox:1.37" &&
  eq "$(jp km -n a07-build get deploy api '{.status.readyReplicas}|{.status.updatedReplicas}|{.status.replicas}')" '2|2|2' &&
  curl -fsS -m 5 "${REG_SCHEME}://${REG}/v2/ckad/a07-api/tags/list" | grep -q '"2.0"' &&
  eq "$(km -n a07-build exec deploy/api -c app -- wget -qO- http://127.0.0.1:8000/)" 'a07-api v2' &&
  km -n a07-build exec deploy/api -c app -- test -f /app/server.py &&
  ! km -n a07-build exec deploy/api -c app -- test -e /app/secrets &&
  grep -qx 'secrets/' "$W/api/.dockerignore"
}

q02() {
  local p='{.spec.schedule}|{.spec.timeZone}|{.spec.concurrencyPolicy}|{.spec.startingDeadlineSeconds}|{.spec.jobTemplate.spec.template.spec.containers[0].image}'
  eq "$(jp km -n a07-batch get cronjob kr-report "$p")" '30 2 * * *|Asia/Seoul|Forbid|300|busybox:1.37' &&
  eq "$(jp km -n a07-batch get cronjob cleanup '{.spec.schedule}')" '*/5 * * * *' &&
  [[ "$(jp km -n a07-batch get cronjob cleanup '{.spec.suspend}')" != true ]]
}

q03() {
  eq "$(jp km -n a07-batch get job etl '{.status.succeeded}')" 1 &&
  eq "$(jp km -n a07-batch get job etl '{.spec.template.spec.initContainers[?(@.name=="shipper")].restartPolicy}')" Always &&
  eq "$(jp km -n a07-batch get job etl '{.spec.template.spec.containers[*].name}')" etl &&
  km -n a07-batch logs job/etl -c shipper | grep -q 'etl done'
}

q04() {
  eq "$(jp km -n a07-deploy get deploy pay '{.spec.template.spec.containers[0].image}')" nginx:1.27-alpine &&
  [[ "$(jp km -n a07-deploy get deploy pay '{.spec.paused}')" != true ]] &&
  eq "$(jp km -n a07-deploy get deploy pay '{.status.readyReplicas}|{.status.updatedReplicas}|{.status.replicas}')" '3|3|3'
}

q05() {
  eq "$(jp km -n a07-deploy get svc web '{.metadata.uid}')" "$(cat "$W/.svc-web-uid")" &&
  eq "$(jp km -n a07-deploy get svc web '{.spec.selector}')" '{"app":"web"}' &&
  eq "$(jp km -n a07-deploy get deploy web-canary '{.spec.template.metadata.labels.app}|{.spec.template.metadata.labels.track}|{.spec.template.spec.containers[0].image}|{.status.readyReplicas}')" \
     'web|canary|nginx:1.28-alpine|1' &&
  eq "$(jp km -n a07-deploy get deploy web-stable '{.status.readyReplicas}')" 4 &&
  (( $(ready_eps km a07-deploy web) == 5 ))
}

q06() {
  helm --kube-context "$MAIN" -n a07-helm status shop | grep -q 'STATUS: deployed' &&
  ! helm --kube-context "$MAIN" -n a07-decoy status shop &&
  eq "$(jp km -n a07-helm get svc shop '{.spec.ports[0].port}')" 8080 &&
  eq "$(jp km -n a07-helm get deploy shop '{.spec.replicas}|{.status.readyReplicas}|{.spec.template.spec.containers[0].image}')" \
     '3|3|nginx:1.28-alpine'
}

q07() {
  eq "$(jp km -n a07-prod get deploy api '{.spec.replicas}|{.status.readyReplicas}')" '4|4' &&
  grep -q 'replicas: 1' "$W/kustomize/base/deployment.yaml" &&
  kubectl kustomize "$W/kustomize/overlays/prod" | grep -q 'replicas: 4'
}

q08() {
  local port
  port=$(jp km -n a07-obs get deploy catalog '{.spec.template.spec.containers[0].readinessProbe.httpGet.port}')
  [[ "$port" == 80 || "$port" == http ]] &&
  eq "$(jp km -n a07-obs get deploy catalog '{.spec.template.spec.containers[0].readinessProbe.httpGet.path}')" / &&
  eq "$(jp km -n a07-obs get svc catalog '{.spec.selector.app}|{.spec.ports[0].port}|{.spec.ports[0].targetPort}')" 'catalog|80|80' &&
  (( $(ready_eps km a07-obs catalog) == 2 ))
}

q09() {
  eq "$(tr -d '\r' < "$W/crash.txt" | sed '/^[[:space:]]*$/d')" 'FATAL code=E-4417 token rejected by upstream'
}

q10() {
  grep -qx 'apiVersion: policy/v1' "$W/legacy/pdb.yaml" &&
  grep -qx 'apiVersion: autoscaling/v2' "$W/legacy/hpa.yaml" &&
  grep -qx 'apiVersion: networking.k8s.io/v1' "$W/legacy/ingress.yaml" &&
  eq "$(jp km -n a07-obs get pdb catalog-pdb '{.spec.minAvailable}|{.spec.selector.matchLabels.app}')" '1|catalog' &&
  eq "$(jp km -n a07-obs get hpa catalog-hpa '{.spec.metrics[0].resource.name}|{.spec.metrics[0].resource.target.type}|{.spec.metrics[0].resource.target.averageUtilization}|{.spec.minReplicas}|{.spec.maxReplicas}')" \
     'cpu|Utilization|70|2|5' &&
  eq "$(jp km -n a07-obs get ingress catalog-ing '{.spec.ingressClassName}|{.spec.rules[0].host}|{.spec.rules[0].http.paths[0].path}|{.spec.rules[0].http.paths[0].pathType}|{.spec.rules[0].http.paths[0].backend.service.name}|{.spec.rules[0].http.paths[0].backend.service.port.number}')" \
     'traefik|catalog.a07.test|/|Prefix|catalog|80'
}

q11() {
  local r='{.spec.template.spec.containers[0].resources.requests.cpu}|{.spec.template.spec.containers[0].resources.requests.memory}|{.spec.template.spec.containers[0].resources.limits.cpu}|{.spec.template.spec.containers[0].resources.limits.memory}'
  eq "$(jp km -n a07-quota get deploy api "$r")" '150m|128Mi|300m|256Mi' &&
  eq "$(jp km -n a07-quota get deploy api '{.spec.replicas}|{.status.readyReplicas}')" '3|3' &&
  eq "$(jp km -n a07-quota get quota team-quota '{.spec.hard.requests\.cpu}|{.spec.hard.limits\.cpu}|{.spec.hard.pods}')" '1|2|5' &&
  eq "$(jp km -n a07-quota get deploy filler '{.spec.template.spec.containers[0].resources.requests.cpu}|{.status.readyReplicas}')" '500m|1'
}

q12() {
  eq "$(jp km -n a07-cfg get deploy orders '{.spec.template.spec.containers[0].env[0].valueFrom.secretKeyRef.name}|{.spec.template.spec.containers[0].env[0].valueFrom.secretKeyRef.key}|{.spec.template.spec.containers[0].envFrom[0].configMapRef.name}')" \
     'orders-db|db-password|orders-cfg' &&
  eq "$(jp km -n a07-cfg get cm orders-cfg '{.data.MODE}|{.data.REGION}|{.data.RETRY}')" 'batch|ap-northeast-2|3' &&
  eq "$(jp km -n a07-cfg get deploy orders '{.status.readyReplicas}')" 1 &&
  eq "$(km -n a07-cfg exec deploy/orders -- printenv DB_PASSWORD)" Or4ers-a07 &&
  eq "$(km -n a07-cfg exec deploy/orders -- printenv REGION)" ap-northeast-2
}

q13() {
  local p='{.spec.serviceAccountName}|{.spec.automountServiceAccountToken}|{.spec.securityContext.runAsNonRoot}|{.spec.containers[0].securityContext.allowPrivilegeEscalation}|{.spec.containers[0].securityContext.readOnlyRootFilesystem}|{.spec.containers[0].securityContext.capabilities.drop[*]}|{.spec.containers[0].command[2]}'
  km -n a07-sec get sa app-runner &&
  eq "$(jp km -n a07-sec get pod hardened "$p")" 'app-runner|false|true|false|true|ALL|touch /tmp/ready && sleep 3600' &&
  pod_ready km a07-sec hardened &&
  eq "$(km -n a07-sec exec hardened -- id -u)" 1000 &&
  km -n a07-sec exec hardened -- test -f /tmp/ready &&
  ! km -n a07-sec exec hardened -- test -e /var/run/secrets/kubernetes.io/serviceaccount/token
}

q14() {
  eq "$(jp ka -n a07-crd get backends.platform.a07.example.com orders-backend '{.spec.tier}|{.spec.replicas}')" 'gold|2' &&
  eq "$(jp ka -n a07-crd get backends.platform.a07.example.com orders-backend '{range .metadata.managedFields[*]}{.apiVersion}{"\n"}{end}' | sort -u)" \
     'platform.a07.example.com/v1'
}

q15() {
  local want got
  want=$(jp km -n a07-net get pod -l app=pay,tier=api '{range .items[*]}{.status.podIP}{"\n"}{end}' | sort)
  got=$(jp km -n a07-net get endpointslices -l kubernetes.io/service-name=pay '{range .items[*].endpoints[*]}{.addresses[0]}{"\n"}{end}' | sort)
  [[ $(printf '%s\n' "$want" | grep -c .) -eq 3 ]] && eq "$got" "$want" &&
  eq "$(jp km -n a07-net get endpointslices -l kubernetes.io/service-name=pay '{.items[0].ports[0].port}')" 80 &&
  eq "$(jp km -n a07-net get pod pay-debug '{.metadata.labels.app}|{.metadata.labels.tier}')" 'pay|debug'
}

q16() {
  eq "$(jp km -n a07-net get netpol deny-all '{.spec.policyTypes[*]}')" 'Ingress Egress' &&
  eq "$(jp km -n a07-net get netpol frontend-egress '{.spec.podSelector.matchLabels.app}|{.spec.policyTypes[*]}')" 'frontend|Egress' &&
  eq "$(jp km -n a07-net get netpol backend-ingress '{.spec.podSelector.matchLabels.app}|{.spec.policyTypes[*]}|{.spec.ingress[*].from[*].podSelector.matchLabels.app}|{.spec.ingress[*].ports[*].port}')" \
     'backend|Ingress|frontend|80' &&
  km -n a07-net exec frontend -- wget -qO- -T 5 http://backend | grep -q 'Welcome to nginx'
}

q17() {
  eq "$(jp km -n a07-net get netpol deny-all '{.spec.policyTypes[*]}')" 'Ingress Egress' &&
  eq "$(jp km -n a07-net get httproute shop-route '{.spec.parentRefs[0].name}|{.spec.parentRefs[0].namespace}|{.spec.hostnames[0]}|{.spec.rules[0].backendRefs[0].name}|{.spec.rules[0].backendRefs[0].port}')" \
     'traefik-gateway|traefik|shop.a07.test|shop|80' &&
  curl -fsS -m 5 -H 'Host: shop.a07.test' "http://${NODE_IP}:30080/" | grep -q 'Welcome to nginx'
}

# --- run ---------------------------------------------------------------
echo "== CKAD mock exam 2 (a07) grading  $(date '+%F %H:%M') =="
check  1  7 "api v2 image (no secrets)"        q01
check  2  7 "CronJob timeZone + resume"        q02
check  3  6 "Job completes w/ native sidecar"  q03
check  4  6 "rollback to known-good"           q04
check  5  5 "canary gets traffic"              q05
check  6  5 "helm upgrade keeps values"        q06
check  7  4 "kustomize overlay fixed"          q07
check  8  6 "readiness port -> endpoints"      q08
check  9  4 "previous container log"           q09
check 10  5 "removed APIs migrated"            q10
check 11  7 "pods fit in ResourceQuota"        q11
check 12  6 "Secret key + env-file ConfigMap"  q12
check 13  7 "hardened pod running"             q13
check 14  5 "CR via non-deprecated version"    q14
check 15  6 "service pay only real pods"       q15
check 16  7 "default deny + DNS egress"        q16
check 17  7 "HTTPRoute + traefik allow"        q17
echo "---------------------------------------------------"
printf 'TOTAL: %d / %d   (PASS %d, FAIL %d)\n' "$SCORE" "$MAX" "$NPASS" "$NFAIL"
if (( SCORE >= 66 )); then
  echo "RESULT: reference line (66) reached"
else
  echo "RESULT: below reference line (66)"
fi
```

> ⚠️ 이 채점 스크립트는 모든 `kubectl`에 `--context`와 `-n`을 명시하므로 kubeconfig의 현재 상태와 무관하게 동작합니다. **시험 중 여러분의 명령은 그렇지 않았다는 것**이 이 모의고사의 첫 번째 함정입니다.

---

## 6. 풀이

### 6-1. 함정 지도

시험을 마친 뒤 먼저 이 표로 "어디에 함정이 있었는지"를 훑고, 아래 풀이를 보세요.

| 문항 | 숨은 함정 | 놓치면 |
|---|---|---|
| 시작 | current-context가 **cka-aux**. 그리고 **`cka-main` context의 기본 네임스페이스가 `a07-decoy`** 로 바뀌어 있어 `use-context cka-main`을 해도 `-n` 없는 명령은 전부 미끼 네임스페이스로 감 | 여러 문항이 엉뚱한 곳에 적용 |
| Q1 | `.dockerignore`의 `*.py`가 `server.py`를 빌드 컨텍스트에서 빼고, CMD는 `serve.py`(오타). **빌드는 성공**하고 Pod만 CrashLoop / 컨테이너가 둘이라 `set image`에 컨테이너 이름 `app` 필요 | Q1 |
| Q2 | `CRON_TZ=Asia/Seoul 30 2 * * *`는 **검증 오류로 거부**. UTC로 환산(`30 17 * * *`)하면 "컨트롤 플레인 시간대에 의존하지 말 것" 위반 / `cleanup`은 `suspend: true` / `a07-batch`가 **cka-aux에도** 있어 잘못된 context에서도 조용히 성공 | Q2 |
| Q3 | 일반 컨테이너인 사이드카는 끝나지 않아 Job이 영원히 Running. Job의 Pod 템플릿은 **변경 불가** → 삭제 후 재생성 | Q3 |
| Q4 | Deployment가 **일시 정지(paused)** 상태라 `rollout undo`가 거부됨 / 인자 없는 `undo`는 직전 리비전(busybox, CrashLoop)으로 감 → `--to-revision` | Q4 |
| Q5 | Service selector에 `track: stable`이 있어 카나리 Pod가 레이블은 맞아도 **트래픽을 못 받음** | Q5 |
| Q6 | `-n` 없이 `helm upgrade`하면 context 기본 네임스페이스(`a07-decoy`)에서 릴리스를 찾음(`--install`을 붙였다면 미끼 네임스페이스에 **새 릴리스**) / `--set`만 주면 이전 값(`service.port=8080`)이 **차트 기본값으로 초기화** → `--reuse-values` | Q6 |
| Q7 | patch가 존재하지 않는 이름 `api-server`를 가리킴 / `a07-prod` 네임스페이스가 없음 | Q7 |
| Q8 | readiness probe가 8080을 봐서 Pod가 Running인데 **Ready가 아님** → EndpointSlice에서 `ready: false` / 미끼 네임스페이스에 같은 이름의 `catalog` | Q8 |
| Q9 | 현재 컨테이너 로그에는 `recovered`만 있음 → `--previous` | Q9 |
| Q10 | apiVersion만 바꾸면 거부: HPA는 `target.averageUtilization` 구조, Ingress는 `service.name`/`port.number` 구조와 **`pathType` 필수** | Q10 |
| Q11 | `kubectl get pods`·`describe deploy`에는 단서가 없고 **ReplicaSet 이벤트**에 `FailedCreate ... must specify ...` / 3×200m은 쿼터 초과로 2개만 뜸 → 150m | Q11 |
| Q12 | Secret 키는 `DB_PASSWORD`인데 Deployment는 `db-password`를 기대 / `--from-file`로 만들면 키 하나(`orders.env`)가 되어 `MODE` 등이 없음 → `--from-env-file` | Q12 |
| Q13 | SA `app-runner`는 **`default` 네임스페이스에만** 있어 Pod 생성 자체가 거부 / `runAsNonRoot`인데 busybox는 root(UID 0)로 실행 / 읽기 전용 루트에서 `touch /tmp/ready` 실패 → `emptyDir`을 `/tmp`에 | Q13 |
| Q14 | `v1beta1`로 만들어도 **경고만 나오고 성공** → 채점은 어느 버전으로 썼는지(managedFields)를 봄 | Q14 |
| Q15 | 셀렉터 `app=pay`가 `pay-debug` Pod까지 잡음 + targetPort 8080 | Q15 |
| Q16 | 이그레스에 backend만 열고 **DNS(53/UDP·TCP)** 를 빠뜨림 → IP로는 되는데 이름으로는 안 됨 | Q16 |
| Q17 | `parentRefs`에 `namespace: traefik` 누락 + backend 포트 8080 + **`deny-all`이 Traefik → shop 인그레스도 막음**(Q16과 같은 네임스페이스) | Q17 |

**권장 순서:** 시작하자마자 `kubectl config get-contexts`로 현재 context와 NAMESPACE 열 확인 → `kubectl config set-context cka-main --namespace=default`로 되돌릴지 결정(문항 조건에 금지가 없으므로 되돌려도 됨) → 배점 큰 Q11·Q13·Q16·Q17 → 나머지.

### 6-2. 문항별 풀이

📖 표시는 허용 문서에서 찾을 페이지입니다. 별칭 `k=kubectl`, `do="--dry-run=client -o yaml"`을 씁니다.

<details><summary>풀이 — 시작. context와 기본 네임스페이스</summary>

```bash
kubectl config get-contexts
```

```
CURRENT   NAME       CLUSTER        AUTHINFO           NAMESPACE
*         cka-aux    kind-cka-aux   kind-cka-aux
          cka-main   kubernetes     kubernetes-admin   a07-decoy
```

```bash
kubectl config set-context cka-main --namespace=default    # 미끼 제거 (권장)
kubectl config use-context cka-main
```

**포인트:** NAMESPACE 열은 평소 비어 있어서 아무도 안 봅니다. 실제 시험 환경도 문항마다 `use-context`만 주므로, **`-n`을 항상 쓰는 습관**이 유일하게 확실한 방어입니다.

📖 Tasks → Access Applications in a Cluster → Configure Access to Multiple Clusters

</details>

<details><summary>풀이 — Q1. 이미지 v2</summary>

```bash
kubectl config use-context cka-main
cd ~/ckad/a07/api
cat Dockerfile .dockerignore
```

두 군데를 고칩니다.

```dockerfile
FROM python:3.12-alpine
WORKDIR /app
COPY server.py .
EXPOSE 8000
USER 10001
CMD ["python", "server.py"]
```

```
.git
*.pyc
secrets/
```

```bash
docker build -t "$REG/ckad/a07-api:2.0" .
docker run --rm "$REG/ckad/a07-api:2.0" ls -la /app          # server.py만, secrets 없음 (빠른 자기 검증)
docker push "$REG/ckad/a07-api:2.0"
k -n a07-build set image deploy/api app="$REG/ckad/a07-api:2.0"
k -n a07-build rollout status deploy/api
k -n a07-build exec deploy/api -c app -- wget -qO- localhost:8000    # a07-api v2
```

**함정 해설:** 원래 Dockerfile도 **빌드는 성공**합니다. `COPY . .`는 `.dockerignore`가 걸러 낸 나머지(Dockerfile 등)만 복사하고, CMD의 파일 이름은 실행할 때야 확인되기 때문입니다. 결과는 `python: can't open file '/app/serve.py'`로 CrashLoop. `.dockerignore`의 `*.py`만 지우고 `COPY . .`를 그대로 두는 것도 가능하지만, 그때도 `secrets/` 규칙이 남아 있어야 비밀 파일이 빠집니다. **명시적 `COPY server.py .`** 가 가장 안전합니다. `set image deploy/api api=...`는 컨테이너 이름이 없어 실패합니다(`unable to find container named "api"`).

📖 Concepts → Containers → Images, Reference → kubectl → `kubectl set image`. Dockerfile 문법은 허용 문서 밖이므로 `docker build --help`와 평소 연습으로.

</details>

<details><summary>풀이 — Q2. CronJob</summary>

```bash
kubectl config use-context cka-main
k -n a07-batch create cronjob kr-report --image=busybox:1.37 --schedule='30 2 * * *' $do -- date > q2.yaml
```

```yaml
spec:
  schedule: 30 2 * * *
  timeZone: Asia/Seoul
  concurrencyPolicy: Forbid
  startingDeadlineSeconds: 300
  jobTemplate:
    ...
```

```bash
k apply -f q2.yaml
k -n a07-batch get cronjob                      # TIMEZONE 열에 Asia/Seoul
k -n a07-batch get cronjob cleanup -o jsonpath='{.spec.suspend}{"\n"}'     # true
k -n a07-batch patch cronjob cleanup -p '{"spec":{"suspend":false}}'
```

**함정 해설:** 스케줄 문자열 안에 `CRON_TZ=`나 `TZ=`를 넣으면 Kubernetes가 **검증 오류로 거부**합니다(공식 문서 CronJob → "Unsupported TimeZone specification"). 시간대는 `.spec.timeZone`(IANA 이름)으로만 지정합니다. UTC로 바꿔 `30 17 * * *`라고 쓰면 지금은 맞아 보여도 kube-controller-manager의 시간대에 따라 결과가 달라지므로 조건 위반입니다. `cleanup`은 `describe`의 `Suspend: True` 한 줄이 원인입니다. 이 네임스페이스는 cka-aux에도 있어서, context를 안 바꾸면 **오류 없이** 엉뚱한 클러스터에 만들어집니다.

📖 Concepts → Workloads → Workload Management → CronJob (Time zones, Concurrency policy, Schedule suspension)

</details>

<details><summary>풀이 — Q3. 끝나지 않는 Job</summary>

```bash
kubectl config use-context cka-main
k -n a07-batch get pod -l job-name=etl             # 1/2 NotReady (etl Completed, shipper Running)
k -n a07-batch get job etl -o yaml > q3.yaml
k -n a07-batch delete job etl                      # 템플릿은 변경 불가 → 지우고 다시
```

`q3.yaml`에서 `status`, `metadata`의 `uid`·`resourceVersion`·`creationTimestamp`, 그리고 **자동 생성된 `spec.selector`와 템플릿의 `controller-uid`·`batch.kubernetes.io/controller-uid`·`job-name` 레이블**을 지운 뒤, `shipper`를 init 컨테이너로 옮깁니다.

```yaml
spec:
  backoffLimit: 0
  template:
    spec:
      restartPolicy: Never
      volumes:
      - name: data
        emptyDir: {}
      initContainers:
      - name: shipper
        image: busybox:1.37
        restartPolicy: Always
        command: ["sh", "-c", "touch /data/out.log; tail -F /data/out.log"]
        volumeMounts:
        - {name: data, mountPath: /data}
      containers:
      - name: etl
        ...                                        # 그대로
```

```bash
k apply -f q3.yaml
k -n a07-batch wait --for=condition=complete job/etl --timeout=90s
k -n a07-batch logs job/etl -c shipper | tail -2   # row 5 loaded / etl done
```

**함정 해설:** Job은 **Pod의 모든 일반 컨테이너가 끝나야** 완료로 봅니다. `tail -F`는 영원히 끝나지 않으니 Job도 끝나지 않습니다. 네이티브 사이드카(init 컨테이너 + `restartPolicy: Always`)는 메인 컨테이너가 끝나면 kubelet이 정리하므로 Job 완료를 막지 않습니다(v1.33부터 stable). 자동 생성 selector·레이블을 지우지 않고 그대로 적용하면 `selector`가 `manualSelector` 없이 지정됐다는 이유로 거부됩니다.

📖 Concepts → Workloads → Pods → Sidecar Containers (Jobs with sidecar containers)

</details>

<details><summary>풀이 — Q4. 롤백</summary>

```bash
kubectl config use-context cka-main
k -n a07-deploy rollout history deploy/pay
```

```
REVISION  CHANGE-CAUSE
1         known-good
2         switch base image
3         hotfix 0.0.0
```

```bash
k -n a07-deploy rollout undo deploy/pay --to-revision=1
#   error: you cannot rollback a paused deployment; resume it first ...
k -n a07-deploy rollout resume deploy/pay
k -n a07-deploy rollout undo deploy/pay --to-revision=1
k -n a07-deploy rollout status deploy/pay
k -n a07-deploy get deploy pay -o wide                     # nginx:1.27-alpine 3/3
```

**함정 해설:** 인자 없는 `rollout undo`는 **직전 리비전(2, busybox)** 으로 가서 CrashLoop이 됩니다. 항상 history를 먼저 보세요. paused 상태에서는 롤백이 거부되고, 설령 다른 방법으로 템플릿을 바꿔도 paused면 롤아웃이 진행되지 않습니다. "이후 변경이 정상적으로 롤아웃될 수 있는 상태" = **resume**. `rollout history --revision=1`로 그 리비전의 이미지를 확인하는 습관도 좋습니다.

📖 Concepts → Workloads → Workload Management → Deployments (Rolling Back, Pausing and Resuming)

</details>

<details><summary>풀이 — Q5. 카나리</summary>

```bash
kubectl config use-context cka-main
k -n a07-deploy get svc web -o jsonpath='{.spec.selector}{"\n"}'       # {"app":"web","track":"stable"}
k -n a07-deploy create deployment web-canary --image=nginx:1.28-alpine --replicas=1 $do > q5.yaml
# selector·template 레이블을 app: web, track: canary 로
k apply -f q5.yaml
k -n a07-deploy scale deploy web-stable --replicas=4
k -n a07-deploy patch svc web --type=json -p '[{"op":"remove","path":"/spec/selector/track"}]'
k -n a07-deploy get endpointslices -l kubernetes.io/service-name=web    # 5개
```

**함정 해설:** 카나리 Pod를 만들고 엔드포인트를 확인하지 않으면 "카나리를 배포했는데 0%"인 상태로 끝납니다. `kubectl edit svc`로 `track` 줄을 지워도 됩니다. `--type=merge`로 `{"selector":{"app":"web"}}`만 보내면 **맵이 병합**되어 `track`이 남으므로 `"track": null`을 함께 보내야 합니다. Service를 `delete` 후 `expose`하면 ClusterIP가 바뀌어 조건 위반(채점은 UID 비교).

📖 Concepts → Cluster Administration → Managing Workloads (Canary deployments)

</details>

<details><summary>풀이 — Q6. Helm 업그레이드</summary>

```bash
kubectl config use-context cka-main
helm list -A | grep shop                                   # 네임스페이스 a07-helm 확인
helm -n a07-helm get values shop                           # USER-SUPPLIED VALUES: service.port: 8080
helm -n a07-helm upgrade shop ~/ckad/a07/charts/shop --reuse-values \
  --set replicaCount=3 --set image.tag=1.28-alpine
helm -n a07-helm get values shop                           # 세 값 모두
k -n a07-helm get svc,deploy shop
```

**함정 해설:** `--set`을 주면서 `--reuse-values`를 빼면 Helm은 **차트 기본값 + 이번 `--set`** 으로 렌더링하므로 `service.port`가 80으로 돌아갑니다(명령은 성공). `--reset-then-reuse-values`도 이 경우 같은 결과입니다. `-n`을 빼면 kubeconfig의 기본 네임스페이스(`a07-decoy`)에서 릴리스를 찾아 `has no deployed releases`가 나고, 여기에 `--install`까지 붙이면 미끼 네임스페이스에 **두 번째 릴리스**가 생깁니다.

📖 helm.sh/docs → Helm Commands → `helm upgrade` (`--reuse-values`), `helm get values`

</details>

<details><summary>풀이 — Q7. Kustomize</summary>

```bash
kubectl config use-context cka-main
k kustomize ~/ckad/a07/kustomize/overlays/prod
#   Error: ... failed to find unique target for patch ... api-server ...
sed -i 's/name: api-server/name: api/' ~/ckad/a07/kustomize/overlays/prod/replicas-patch.yaml
k kustomize ~/ckad/a07/kustomize/overlays/prod | grep -E 'namespace|replicas'
k create namespace a07-prod
k apply -k ~/ckad/a07/kustomize/overlays/prod
k -n a07-prod get deploy api
```

**함정 해설:** patch는 **이름이 같은 리소스**를 대상으로 합니다(namePrefix가 있다면 접두사 붙기 전 이름). 렌더링이 성공해도 `namespace: a07-prod`는 네임스페이스를 만들어 주지 않으므로 `namespaces "a07-prod" not found`. 네임스페이스를 overlay의 리소스로 추가(`namespace.yaml`)해도 정답입니다.

📖 Tasks → Manage Kubernetes Objects → Declarative Management of Kubernetes Objects Using Kustomize (Patches)

</details>

<details><summary>풀이 — Q8. 엔드포인트 없는 Service</summary>

```bash
kubectl config use-context cka-main
k -n a07-obs get pod -l app=catalog                       # Running 이지만 READY 0/1
k -n a07-obs get endpointslices -l kubernetes.io/service-name=catalog -o yaml | grep -A2 conditions   # ready: false
k -n a07-obs describe pod -l app=catalog | grep -A2 Readiness
#   Readiness probe failed: Get "http://10.244.x.x:8080/": ... connection refused
k -n a07-obs patch deploy catalog --type=json \
  -p '[{"op":"replace","path":"/spec/template/spec/containers/0/readinessProbe/httpGet/port","value":80}]'
k -n a07-obs rollout status deploy/catalog
```

**함정 해설:** 셀렉터는 맞으므로 EndpointSlice에 **주소는 있지만 `ready: false`** 입니다. `kubectl get endpoints`(폐기 예정 API, 경고 출력)나 `describe svc`의 Endpoints 줄은 Ready 주소만 보여서 "비어 있음"으로 보입니다. 셀렉터·targetPort를 의심해 Service를 고치면 조건 위반. `-n` 없이 `kubectl edit deploy catalog`를 하면 **미끼 네임스페이스의 같은 이름 Deployment**를 고치게 됩니다.

📖 Tasks → Configure Pods and Containers → Configure Liveness, Readiness and Startup Probes, Tasks → Debug → Debug Applications → Debug Services

</details>

<details><summary>풀이 — Q9. 이전 컨테이너의 로그</summary>

```bash
kubectl config use-context cka-main
k -n a07-obs get pod batch-runner                          # RESTARTS 1
k -n a07-obs logs batch-runner                             # recovered ... (현재 컨테이너)
k -n a07-obs logs batch-runner --previous | grep '^FATAL' > ~/ckad/a07/crash.txt
cat ~/ckad/a07/crash.txt
```

**함정 해설:** 현재 컨테이너의 로그에는 원인이 없습니다. `--previous`(`-p`)는 **같은 Pod 안의 직전 컨테이너 인스턴스** 로그를 보여 줍니다. Pod가 다시 만들어졌다면(Deployment 롤아웃 등) 이전 Pod의 로그는 사라지므로, 원인 조사 전에 Pod를 지우지 마세요. `kubectl describe pod`의 `Last State: Terminated, Exit Code: 1`이 "이전 로그를 볼 차례"라는 신호입니다.

📖 Reference → kubectl → `kubectl logs` (`--previous`), Tasks → Debug → Debug Applications → Debug Running Pods

</details>

<details><summary>풀이 — Q10. 폐기된 API</summary>

```bash
kubectl config use-context cka-main
k api-resources | grep -E 'poddisruptionbudgets|horizontalpodautoscalers|ingresses'
k explain hpa.spec.metrics.resource.target
```

```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
...                                   # 나머지 그대로
---
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: catalog-hpa
  namespace: a07-obs
spec:
  scaleTargetRef: {apiVersion: apps/v1, kind: Deployment, name: catalog}
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: catalog-ing
  namespace: a07-obs
spec:
  ingressClassName: traefik
  rules:
  - host: catalog.a07.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: catalog
            port:
              number: 80
```

```bash
k apply -f ~/ckad/a07/legacy/
k -n a07-obs get pdb,hpa,ingress
```

**함정 해설:** 세 API는 모두 오래전에 제거되었습니다(`policy/v1beta1` PDB와 `autoscaling/v2beta1` HPA는 v1.25, Ingress `v1beta1`은 v1.22). PDB는 apiVersion만 바꾸면 되지만 HPA의 `targetAverageUtilization`과 Ingress의 `serviceName`/`servicePort`는 **구조가 바뀌어** 그대로 두면 strict 검증에서 "unknown field"로 거부됩니다. Ingress v1은 `pathType`이 필수입니다. `k apply -f <디렉터리>`로 세 파일을 한 번에.

📖 Reference → API Overview → Deprecated API Migration Guide (v1.25, v1.22 절)

</details>

<details><summary>풀이 — Q11. 생기지 않는 Pod</summary>

```bash
kubectl config use-context cka-main
k -n a07-quota get deploy,rs,pod                          # deploy 0/3, rs DESIRED 3 CURRENT 0, Pod 없음
k -n a07-quota describe rs -l app=api | tail -3
#   Warning  FailedCreate  ...  Error creating: pods "api-..." is forbidden: failed quota: team-quota:
#            must specify limits.cpu for: api; limits.memory for: api; requests.cpu for: api; requests.memory for: api
k -n a07-quota describe quota team-quota                  # requests.cpu 500m/1 (filler)
```

계산: 남은 `requests.cpu` = 1000m − 500m = 500m → 3개면 개당 ≤ 166m → **50m 단위 최댓값 150m**. 나머지 한도도 확인합니다: `limits.cpu` 1000m + 3×300m = 1900m ≤ 2, `requests.memory` 256Mi + 3×128Mi = 640Mi ≤ 1Gi, `limits.memory` 512Mi + 3×256Mi = 1280Mi ≤ 2Gi, `pods` 1 + 3 = 4 ≤ 5.

```bash
k -n a07-quota set resources deploy api --requests=cpu=150m,memory=128Mi --limits=cpu=300m,memory=256Mi
k -n a07-quota rollout status deploy/api
k -n a07-quota describe quota team-quota                  # requests.cpu 950m/1
```

**함정 해설:** 쿼터 거부는 **Pod 생성 단계**에서 일어나므로 Pod 이벤트가 아니라 **ReplicaSet 이벤트**에만 남습니다. `describe deploy`의 Conditions에는 `ReplicaFailure`가 보이긴 하지만 자세한 이유는 RS에 있습니다. 200m으로 맞추면 오류 없이 **2개만** 뜨고 세 번째는 `exceeded quota`로 계속 실패합니다. LimitRange로 기본값을 넣는 방법은 "컨테이너 리소스를 이렇게" 조건과 다르므로 쓰지 않습니다.

📖 Concepts → Policies → Resource Quotas, Reference → kubectl → `kubectl set resources`

</details>

<details><summary>풀이 — Q12. 설정이 없는 Deployment</summary>

```bash
kubectl config use-context cka-main
k -n a07-cfg get pod -l app=orders                        # CreateContainerConfigError
k -n a07-cfg describe pod -l app=orders | tail -3
#   Error: configmap "orders-cfg" not found     (이것을 고치면 다음 오류)
#   Error: couldn't find key db-password in Secret a07-cfg/orders-db
k -n a07-cfg get secret orders-db -o jsonpath='{.data.DB_PASSWORD}' | base64 -d; echo
k -n a07-cfg create secret generic orders-db \
  --from-literal=db-password="$(k -n a07-cfg get secret orders-db -o jsonpath='{.data.DB_PASSWORD}' | base64 -d)" \
  --dry-run=client -o yaml | k apply -f -
k -n a07-cfg create configmap orders-cfg --from-env-file=$HOME/ckad/a07/orders.env
k -n a07-cfg get pod -l app=orders -w                     # 잠시 후 Running (kubelet이 재시도)
k -n a07-cfg exec deploy/orders -- env | grep -E 'MODE|REGION|RETRY|DB_'
```

**함정 해설:** `apply`로 덮어쓴 Secret에는 `db-password`만 남고 원래 키 `DB_PASSWORD`는 사라집니다(조건상 문제없음, 남기고 싶다면 `patch`로 키를 추가). `--from-file=orders.env`는 키 하나(`orders.env`)에 파일 전체가 들어가 `envFrom`이 `MODE` 등을 만들지 못합니다. `--from-env-file`은 `#` 주석 줄을 건너뜁니다. CreateContainerConfigError는 **kubelet이 계속 재시도**하므로 Deployment를 재시작하지 않아도 풀립니다.

📖 Tasks → Configure Pods and Containers → Configure a Pod to Use a ConfigMap (Create ConfigMaps from env files)

</details>

<details><summary>풀이 — Q13. 하드닝 Pod</summary>

```bash
kubectl config use-context cka-main
k apply -f ~/ckad/a07/hardened.yaml
#   Error from server (Forbidden): ... error looking up service account a07-sec/app-runner ...
k get sa -A | grep app-runner                             # default 네임스페이스에만
k -n a07-sec create serviceaccount app-runner
```

파일을 고칩니다.

```yaml
spec:
  serviceAccountName: app-runner
  automountServiceAccountToken: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  volumes:
  - name: tmp
    emptyDir: {}
  containers:
  - name: app
    image: busybox:1.37
    command: ["sh", "-c", "touch /tmp/ready && sleep 3600"]
    volumeMounts:
    - {name: tmp, mountPath: /tmp}
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

```bash
k apply -f ~/ckad/a07/hardened.yaml
k -n a07-sec get pod hardened                             # Running
k -n a07-sec exec hardened -- sh -c 'id -u; ls /tmp; ls /var/run/secrets 2>&1'
```

**함정 해설:** 오류가 **세 겹**입니다. ① SA가 없으면 Pod 생성 자체가 거부(이 오류만 보입니다), ② SA를 만들면 이번엔 `CreateContainerConfigError: container has runAsNonRoot and image will run as root` (busybox는 USER가 없어 UID 0), ③ UID를 주면 이번엔 `touch: /tmp/ready: Read-only file system`으로 CrashLoop. 하나씩 고칠 때마다 **다음 오류를 기대하고** 확인하세요. `runAsNonRoot`를 지우거나 `readOnlyRootFilesystem: false`로 바꾸면 조건 위반입니다.

📖 Tasks → Configure Pods and Containers → Configure a Security Context for a Pod or Container, Configure Service Accounts for Pods (Opt out of API credential automounting)

</details>

<details><summary>풀이 — Q14. CRD 버전</summary>

```bash
kubectl config use-context cka-aux
k get crd backends.platform.a07.example.com -o jsonpath='{range .spec.versions[*]}{.name} served={.served} storage={.storage} deprecated={.deprecated}{"\n"}{end}'
#   v1beta1 served=true storage=false deprecated=true
#   v1 served=true storage=true deprecated=
k explain backend.spec --api-version=platform.a07.example.com/v1
cat <<'EOF' | k apply -f -
apiVersion: platform.a07.example.com/v1
kind: Backend
metadata:
  name: orders-backend
  namespace: a07-crd
spec:
  tier: gold
  replicas: 2
EOF
```

**함정 해설:** `kubectl explain backend`가 보여 주는 예시나 `k get crd -o yaml`의 **첫 번째 버전**(`v1beta1`)을 그대로 쓰면, `Warning: ... is deprecated` 경고만 나오고 **성공**합니다. 경고를 읽는 습관이 핵심입니다. 잘못 만들었다면 지우고 `v1`로 다시 만들어야 합니다(같은 객체를 v1로 `apply`하면 managedFields에 v1beta1 기록이 남을 수 있음).

📖 Tasks → Extend Kubernetes → Use Custom Resources → Versions in CustomResourceDefinitions

</details>

<details><summary>풀이 — Q15. 엉뚱한 Pod를 잡는 Service</summary>

```bash
kubectl config use-context cka-main
k -n a07-net get pod -l app=pay --show-labels             # pay-xxx 3개(tier=api) + pay-debug(tier=debug)
k -n a07-net get endpointslices -l kubernetes.io/service-name=pay -o wide   # 4개
k -n a07-net patch svc pay --type=merge \
  -p '{"spec":{"selector":{"app":"pay","tier":"api"},"ports":[{"port":80,"targetPort":"http"}]}}'
k -n a07-net get endpointslices -l kubernetes.io/service-name=pay -o wide   # 3개
```

**함정 해설:** "가끔만 응답"은 **엔드포인트 일부가 잘못된 대상**이라는 신호입니다(여기선 포트 8080 자체도 틀려 사실상 전혀 응답하지 않지만, targetPort만 고치면 4번 중 1번은 실패). `pay-debug`의 레이블을 바꾸면 조건 위반. Deployment의 selector(`app=pay,tier=api`)를 그대로 Service에 쓰는 것이 가장 확실합니다.

📖 Concepts → Services, Load Balancing, and Networking → Service

</details>

<details><summary>풀이 — Q16. 기본 거부 아래의 통신</summary>

```bash
kubectl config use-context cka-main
cat <<'EOF' | k apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-egress
  namespace: a07-net
spec:
  podSelector:
    matchLabels: {app: frontend}
  policyTypes: ["Egress"]
  egress:
  - to:
    - podSelector:
        matchLabels: {app: backend}
    ports:
    - {protocol: TCP, port: 80}
  - to:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: kube-system}
      podSelector:
        matchLabels: {k8s-app: kube-dns}
    ports:
    - {protocol: UDP, port: 53}
    - {protocol: TCP, port: 53}
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-ingress
  namespace: a07-net
spec:
  podSelector:
    matchLabels: {app: backend}
  policyTypes: ["Ingress"]
  ingress:
  - from:
    - podSelector:
        matchLabels: {app: frontend}
    ports:
    - {protocol: TCP, port: 80}
EOF
k -n a07-net exec frontend -- wget -qO- -T 3 http://backend | head -4
```

**함정 해설:** DNS 규칙을 빼면 `wget: bad address 'backend'`. 백엔드 Pod IP로 직접 해 보면 되기 때문에 "정책은 맞는데 이상하다"며 시간을 씁니다. **IP로는 되고 이름으로는 안 되면 DNS 이그레스**. DNS 규칙의 `namespaceSelector`와 `podSelector`를 **한 항목(AND)** 으로 묶었는지 확인하세요. 대시(`-`)를 하나 더 붙이면 OR가 되어 "kube-system 전체 또는 모든 네임스페이스의 kube-dns 레이블 Pod"가 됩니다. 이그레스가 막힌 Pod에서 Service ClusterIP로 나가는 트래픽은 실제 백엔드 Pod로 변환된 뒤 정책이 적용되므로 `to.podSelector: app=backend`로 충분합니다.

📖 Concepts → Services, Load Balancing, and Networking → Network Policies (Targeting a namespace by its name, 기본 정책 예시)

</details>

<details><summary>풀이 — Q17. 붙지 않는 HTTPRoute</summary>

```bash
kubectl config use-context cka-main
k -n a07-net describe httproute shop-route | sed -n '/Status/,$p'
#   Parents: 비어 있음 (Gateway a07-net/traefik-gateway 를 찾지 못함)
k -n a07-net patch httproute shop-route --type=json -p '[
  {"op":"add","path":"/spec/parentRefs/0/namespace","value":"traefik"},
  {"op":"replace","path":"/spec/rules/0/backendRefs/0/port","value":80}]'
k -n a07-net get httproute shop-route -o jsonpath='{range .status.parents[0].conditions[*]}{.type}={.status} {end}{"\n"}'
#   Accepted=True ResolvedRefs=True
curl -s -m 5 -H 'Host: shop.a07.test' http://192.168.56.11:30080/ -o /dev/null -w '%{http_code}\n'
#   504 (또는 시간 초과) ← Traefik이 shop Pod에 닿지 못함: deny-all
cat <<'EOF' | k apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: shop-from-traefik
  namespace: a07-net
spec:
  podSelector:
    matchLabels: {app: shop}
  policyTypes: ["Ingress"]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: traefik}
    ports:
    - {protocol: TCP, port: 80}
EOF
curl -s -H 'Host: shop.a07.test' http://192.168.56.11:30080/ | grep title     # Welcome to nginx!
```

**함정 해설:** 세 가지가 겹쳐 있습니다. ① `parentRefs.namespace` 누락(A06 Q17과 같은 실수), ② backend 포트가 Service 포트(80)가 아닌 8080 → `ResolvedRefs=False`, ③ 이 두 개를 고쳐 status가 모두 True가 되어도 **데이터 경로**는 `deny-all`에 막혀 있습니다. Route의 status는 "설정이 받아들여졌다"는 뜻이지 "트래픽이 간다"는 뜻이 아닙니다. 끝까지 `curl`로 확인하세요. `deny-all`을 지우면 조건 위반.

📖 kubernetes.io/docs/concepts/services-networking/gateway/ (HTTPRoute의 parentRefs·backendRefs), Concepts → Network Policies. CKAD에서는 gateway-api.sigs.k8s.io를 열 수 없습니다(A00 참고).

</details>

---

## 7. 결과 해석

### 7-1. 점수대별 다음 행동

| 점수 | 해석 | 다음 행동 |
|---|---|---|
| 85~100 | 함정까지 읽어 냄 | 실제 응시 준비 완료에 가까움. killer.sh 시뮬레이터 2회로 마무리 |
| 66~84 | 참고선 통과 | 6-1 함정 지도에서 놓친 줄만 다시. 같은 셋업으로 **90분** 재도전 |
| 45~65 | 지식은 있으나 조건·상태 확인이 약함 | A06을 90분에 다시 → A07을 다시. 매 문항 끝에 **검증 명령** 한 줄을 습관화 |
| 0~44 | 기본기부터 | A06 7-2 표의 편을 다시 보고 A06 66점 이상 후 재도전 |

### 7-2. 틀린 문항 → 다시 볼 편

| 틀린 문항 | 도메인 | 다시 볼 편 |
|---|---|---|
| Q1, Q2, Q3 | 설계·빌드 | [A01 애플리케이션 설계·빌드](./a01-design-build.md) |
| Q4, Q5, Q6, Q7 | 배포 | [A02 애플리케이션 배포](./a02-deployment.md) |
| Q8, Q9, Q10 | 관측성·유지보수 | [A03 관측성·유지보수](./a03-observability-maintenance.md) |
| Q11, Q12, Q13, Q14 | 환경·설정·보안 | [A04 환경·설정·보안](./a04-environment-config-security.md) |
| Q15, Q16, Q17 | 서비스·네트워킹 | [A05 서비스·네트워킹](./a05-services-networking.md) |
| 여러 문항이 `a07-decoy`에 | 시험 습관 | [A00 시험 요령](./a00-exam-overview-lab.md) — 과제 시작 루틴 |

> 💡 A06에서 맞힌 주제를 A07에서 틀렸다면 지식이 아니라 **"확인" 습관** 문제입니다. 이 모의고사의 함정은 거의 모두 "명령은 성공했다"는 공통점이 있습니다. 성공 메시지 대신 **결과(엔드포인트 수, Ready 수, status 조건, 실제 curl)** 를 보는 것이 CKAD의 마지막 관문입니다.

---

## 트러블슈팅

| 증상 | 원인 | 해결 |
|---|---|---|
| 셋업이 `[*] building ...`에서 멈춤 | `python:3.12-alpine` pull 중이거나 레지스트리 push 실패 | `docker pull python:3.12-alpine` 확인, Podman이면 `PUSH_OPTS=--tls-verify=false` |
| 셋업이 `helm release shop`에서 시간 초과 | 이미지 pull이 느림 | `kubectl --context cka-main -n a07-helm get pod` 확인 후 재실행(`upgrade --install`이라 안전) |
| 셋업 후 `pay` 리비전이 1~3이 아님 | 셋업을 두 번 실행해 리비전이 누적됨 | `ckad-clean`으로 되돌린 뒤 한 번만 실행 |
| Q1 새 Pod가 계속 옛 이미지 | 같은 태그를 다시 push하고 `imagePullPolicy: IfNotPresent` | 태그를 바꾸는 것이 원칙(문항은 2.0 고정). 재빌드했다면 `rollout restart`와 함께 `imagePullPolicy: Always` 검토 |
| Q16 정책이 맞는데 DNS만 실패 | CoreDNS Pod 레이블이 `k8s-app=kube-dns`가 아닌 환경 | `kubectl -n kube-system get pod --show-labels \| grep dns` |
| Q17 curl이 404 | Host 헤더 오타 또는 HTTPRoute가 아직 Accepted 아님 | `kubectl -n a07-net describe httproute shop-route` |
| 시험 후 다른 편 과제가 이상하게 동작 | `cka-main` context의 기본 네임스페이스가 `a07-decoy`로 남아 있음 | `kubectl config set-context cka-main --namespace=default` |

---

## 핵심 정리

- 시작 직후 **`kubectl config get-contexts`의 CURRENT와 NAMESPACE 열** 두 가지를 봅니다. 그리고 모든 명령에 `-n`.
- **명령 성공 ≠ 과제 완료.** 빌드 성공 후 CrashLoop, `CRON_TZ` 대신 `timeZone`, `--reuse-values` 누락, 폐기 예정 CRD 버전 경고, Accepted인데 막힌 Route — 확인은 항상 결과로.
- Pod가 **아예 없으면** ReplicaSet 이벤트(쿼터·어드미션), Pod가 **Running인데 트래픽이 없으면** readiness와 EndpointSlice의 `ready`, Pod가 **재시작 중이면** `logs --previous`.
- Job의 사이드카는 **네이티브 사이드카**로. 일반 컨테이너 사이드카는 Job을 영원히 Running으로 둡니다.
- 기본 거부 NetworkPolicy가 있으면 **DNS 이그레스**와 **Ingress 컨트롤러(Traefik)로부터의 인그레스**를 함께 떠올립니다.

---

## ⏭️ 다음 단계

A06·A07을 66점 이상으로 통과했다면 CKAD 실기 준비는 마무리 단계입니다. 응시 직전에는 등록 시 제공되는 **killer.sh 시뮬레이터 2회**로 실제 시험 UI와 난이도에 익숙해지세요.

**Kubestronaut 완성 방향.** CNCF의 Kubestronaut는 **CKA·CKAD·CKS·KCNA·KCSA** 다섯 자격을 모두 유효한 상태로 가진 사람에게 주어집니다. 이 시리즈 기준으로 남은 길은 다음과 같습니다.

| 자격 | 형식 | 이 시리즈에서 |
|---|---|---|
| CKA | 실기 | [CKA 탭](../cka/k00-exam-overview-lab.md) K00~K08 |
| CKAD | 실기 | 이 탭 A00~A07 (지금 여기) |
| CKS | 실기, **유효한 CKA 필요** | [CKS 탭](../cks/s00-exam-overview-lab.md) S00~S08 — A07 Q13·Q16의 securityContext·NetworkPolicy가 출발점 |
| KCNA | 객관식(입문) | 본편 Chapter 2·5와 CKA·CKAD 탭이면 범위 대부분을 덮음. 클라우드 네이티브 생태계·관측성·GitOps 용어 정리 위주 |
| KCSA | 객관식(보안 입문) | 본편 보안 탭(Day 66~73)과 CKS 탭의 개념 부분 |

실기 셋을 먼저 끝내고 객관식 둘을 묶어서 보는 순서가 일반적입니다. 자격마다 유효 기간이 있으므로 **다섯 개가 동시에 유효한 기간**이 생기도록 일정을 잡으세요. 응시 조건·유효 기간·프로그램 혜택은 바뀔 수 있으니 **CNCF·Linux Foundation 공식 페이지에서 반드시 직접 확인**하세요. 이 탭과 마찬가지로, 유출 문제가 아닌 공식 커리큘럼 기반 연습만이 실력이 됩니다.
