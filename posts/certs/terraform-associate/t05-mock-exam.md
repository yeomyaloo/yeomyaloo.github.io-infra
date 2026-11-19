---
title: "Terraform Associate T05. 모의고사 (57문항 · 60분)"
date: 2026-11-19T22:35:17+09:00
weight: 6
tags: ["terraform-associate", "certification", "terraform", "mock-exam"]
summary: "[직접 만든 창작 문항 · 기출·덤프 아님] 목표 1~8을 섞은 57문항을 60분 안에 푼다. 참과 거짓, 단일 선택, 복수 선택 형식에 익숙해진다. 목표별 채점표로 약한 목표를 찾고, 다시 볼 편(T01~T04)으로 연결한다."
image: ""
draft: false
---
> **자격증 준비 · Terraform Associate** | 난이도 ★★★☆☆ | 예상 소요 시험 60분 + 채점·복기 1시간

> ⚠️ **이 모의고사의 57문항은 모두 HashiCorp가 공개한 Terraform Associate (004) 시험 목표 8개를 바탕으로 이 시리즈가 직접 창작한 연습 문제입니다.** 실제 시험 문제나 유출 문제(덤프)가 아니며, 그런 자료를 옮기거나 재구성하지도 않았습니다. 리소스 이름·값·상황은 모두 새로 지었습니다. "실제 기출"이라고 주장하는 자료는 응시 규정 위반이니 믿지도, 퍼뜨리지도 마세요.

## 🎯 오늘의 목표

- 목표 1~8을 섞은 57문항을 **60분 안에** 푼다
- 참과 거짓, 단일 선택, 복수 선택 형식에 익숙해진다
- 목표별 채점표로 약한 목표를 찾고, 다시 볼 편(T01~T04)으로 연결한다

---

## 1. 시험 안내

| 항목 | 이 모의고사 | 실제 시험 (2026-11-19 공식 페이지 확인) |
|---|---|---|
| 버전 | 004 목표 기준, Terraform 1.12 | Terraform Associate (004), Terraform 1.12 |
| 시간 | **60분** | 1시간, 온라인 감독 |
| 문항 수 | **57문항**, 문항당 1점 | 공식 미공개 (교육 업체들이 57문항 안팎을 언급) |
| 형식 | 참과 거짓 10, 단일 선택 38, 복수 선택 9 | 참과 거짓 / 단일 선택 / 복수 선택 (고를 개수 표시) |
| 합격 참고선 | **40점(70%)** | 공식 미공개 |
| 채점 | 복수 선택은 **모두 맞아야** 1점 | 공식 채점 방식 미공개 |

**푸는 방법**

1. 타이머를 60분에 맞추고, 문서·검색·터미널 없이 풉니다.
2. 답은 종이나 메모장에 `1-C, 2-참, 3-AC …` 형태로 적습니다.
3. 2분 넘게 걸리는 문항은 번호에 표시하고 넘어갔다가 마지막에 돌아옵니다.
4. 끝나면 **5장 정답표**로 채점하고, **6장 목표별 채점표**에 틀린 개수를 적은 뒤 **7장 해설**을 봅니다.

---

## 2. 문항 1~20

**1.** Terraform 설정이 "선언형"이라는 말의 의미로 옳은 것은?
- A. 사용자가 실행할 API 호출의 순서를 직접 적는다
- B. apply할 때마다 모든 리소스를 새로 만든다
- C. 원하는 최종 상태를 적으면 Terraform이 필요한 작업과 순서를 계산한다
- D. 설정 파일은 반드시 JSON으로 써야 한다

**2.** (참과 거짓) `terraform plan`은 실제 인프라를 변경하지 않는다.

**3.** provider에 대한 설명으로 옳은 것은? **(두 개 선택)**
- A. `terraform init` 때 플러그인으로 설치된다
- B. provider 버전은 `provider` 블록의 `version` 인수로 지정하는 것이 권장된다
- C. 하나의 설정에서 여러 provider를 함께 쓸 수 있다
- D. provider는 항상 Terraform CLI와 같은 버전 번호로 함께 릴리스된다

**4.** 변수 `zone`에 대해 환경 변수 `TF_VAR_zone=env`, `terraform.tfvars`에 `zone = "file"`, 그리고 `terraform plan -var-file=custom.tfvars`(안에 `zone = "custom"`)를 실행했다. 사용되는 값은?
- A. `env`
- B. `file`
- C. `custom`
- D. 값이 여러 곳에 있어 오류가 난다

**5.** local backend를 쓰면서 state 파일 위치를 `state/dev.tfstate`로 바꾸려 한다. 알맞은 방법은?
- A. `terraform { backend "local" { path = "state/dev.tfstate" } }`
- B. `terraform plan -state-dir=state`
- C. 환경 변수 `TF_STATE_PATH=state/dev.tfstate`
- D. local backend는 경로를 바꿀 수 없다

**6.** `required_providers`에 새 provider를 추가한 직후 `terraform plan`을 실행했더니 provider가 설치되지 않았다는 오류가 났다. 먼저 실행할 명령은?
- A. `terraform init`
- B. `terraform validate`
- C. `terraform providers`
- D. `terraform get`

**7.** 저장소의 기본 브랜치에 커밋이 들어오면 HCP Terraform이 자동으로 run을 시작하게 하려 한다. 알맞은 워크플로는?
- A. API 기반
- B. CLI 기반
- C. VCS 기반
- D. Local 실행 모드

**8.** (참과 거짓) 하나의 resource 블록에 `count`와 `for_each`를 함께 써서 인스턴스를 만들 수 있다.

**9.** 설정 파일 안의 `import` 블록으로 기존 리소스를 가져오는 기능이 처음 도입된 Terraform 버전은?
- A. 0.13
- B. 1.0
- C. 1.5
- D. 1.12

**10.** 루트 모듈에서 `module "network"`의 출력 `vpc_id`를 참조하는 올바른 표현은?
- A. `module.network.outputs.vpc_id`
- B. `module.network.vpc_id`
- C. `var.network.vpc_id`
- D. `output.network.vpc_id`

**11.** Terraform 하나로 여러 클라우드와 SaaS를 함께 관리할 때의 이점은? **(두 개 선택)**
- A. 하나의 언어와 워크플로로 모든 대상을 다룬다
- B. 서로 다른 provider의 리소스 사이 의존성을 한 설정에서 표현할 수 있다
- C. 모든 클라우드의 리소스 타입 이름과 인수가 같아진다
- D. 클라우드 비용이 자동으로 최적화된다

**12.** (참과 거짓) 상태 잠금은 사용자가 `terraform lock` 명령을 먼저 실행해야 걸린다.

**13.** `.terraform.lock.hcl`에 기록되지 **않는** 것은?
- A. 선택된 provider 버전
- B. provider 패키지 해시
- C. 모듈 버전
- D. provider 버전 제약

**14.** `terraform fmt`로 현재 디렉터리뿐 아니라 하위 디렉터리의 `.tf` 파일까지 정리하려면?
- A. `-check`
- B. `-diff`
- C. `-recursive`
- D. `-all`

**15.** `sensitive = true`로 선언한 변수 값이 **평문으로 남을 수 있는** 곳은? **(두 개 선택)**
- A. state 파일
- B. `terraform plan -out`으로 저장한 plan 파일
- C. `terraform plan`의 화면 출력
- D. `terraform apply`의 화면 출력

**16.** HCP Terraform 변수 세트를 적용할 수 있는 범위는? **(세 개 선택)**
- A. 조직 전체(전역)
- B. 프로젝트
- C. 선택한 워크스페이스들
- D. 한 사용자의 로컬 PC

**17.** 원격 backend를 쓰는 이유로 옳은 것은? **(세 개 선택)**
- A. 팀원과 같은 state를 공유한다
- B. backend에 따라 상태 잠금을 쓸 수 있다
- C. state 안의 민감 값이 자동으로 지워진다
- D. 저장소의 암호화와 접근 제어를 활용할 수 있다

**18.** 검토한 plan과 **정확히 같은 변경만** 적용되게 하려면?
- A. `terraform apply -auto-approve`
- B. `terraform plan -out=tfplan` 후 `terraform apply tfplan`
- C. `terraform plan` 후 `terraform apply -refresh=false`
- D. `terraform validate` 후 `terraform apply`

**19.** `data` 블록으로 조회만 하던 네트워크는 `terraform destroy` 때 어떻게 되나?
- A. 다른 리소스와 함께 삭제된다
- B. 삭제되지 않는다
- C. destroy가 오류로 멈춘다
- D. state에 import된다

**20.** `TF_LOG`에 줄 수 있는 값 중 가장 자세한 로그 수준은?
- A. `DEBUG`
- B. `INFO`
- C. `TRACE`
- D. `ALL`

---

## 3. 문항 21~40

**21.** 레지스트리 모듈을 1.x 범위 안의 최신 버전까지만 허용하려 한다(2.0은 제외). 알맞은 `version`은?
- A. `">= 1.0"`
- B. `"~> 1.0"`
- C. `"~> 1.0.0"`
- D. `"1.*"`

**22.** (참과 거짓) Terraform은 state 없이도 매번 클라우드의 모든 리소스를 조회해 어떤 것이 자신이 만든 리소스인지 정확히 판단한다.

**23.** HCP Terraform에서 OPA 정책을 작성하는 언어는?
- A. Sentinel
- B. HCL
- C. Rego
- D. Python

**24.** `[for s in ["a", "b", "c"] : upper(s) if s != "b"]`의 결과는?
- A. `["A", "B", "C"]`
- B. `["A", "C"]`
- C. `["a", "c"]`
- D. `{ a = "A", c = "C" }`

**25.** backend 설정을 바꾸면서 기존 state를 옮기지 **않고** 새 설정으로만 초기화하려 한다. 알맞은 옵션은?
- A. `terraform init -migrate-state`
- B. `terraform init -reconfigure`
- C. `terraform init -upgrade`
- D. `terraform init -backend=false`

**26.** `terraform plan -out=tfplan`으로 저장한 계획을 사람이 읽을 수 있게 보려면?
- A. `terraform show tfplan`
- B. `terraform state show tfplan`
- C. `cat tfplan`
- D. `terraform output tfplan`

**27.** `dynamic "setting" { for_each = var.settings  content { ... } }`의 `content` 안에서 현재 원소의 값을 가리키는 표현은?
- A. `each.value`
- B. `setting.value`
- C. `dynamic.value`
- D. `count.index`

**28.** `terraform import` 명령보다 `import` 블록이 나은 점은? **(두 개 선택)**
- A. plan으로 가져올 대상을 미리 확인할 수 있다
- B. `-generate-config-out`으로 resource 블록 초안을 만들 수 있다
- C. state 파일이 없어도 동작한다
- D. provider가 import를 지원하지 않아도 동작한다

**29.** (참과 거짓) IaC를 쓰면 인프라 변경을 코드 리뷰와 버전 관리 이력으로 추적할 수 있다.

**30.** 루트 모듈에 `provider "aws" { alias = "west" ... }`가 있다. 자식 모듈 `module "dr"`이 이 별칭 설정을 기본 `aws` provider로 쓰게 하려면?
- A. `module` 블록에 `provider = aws.west`
- B. `module` 블록에 `providers = { aws = aws.west }`
- C. 자식 모듈의 변수로 provider를 넘긴다
- D. 자식 모듈은 별칭 provider를 쓸 수 없다

**31.** `lifecycle { prevent_destroy = true }`가 있는 리소스를 삭제하는 plan을 만들면?
- A. 경고 없이 삭제된다
- B. 오류로 plan이 실패한다
- C. 경고만 내고 삭제한다
- D. state에서만 제거된다

**32.** (참과 거짓) 자식 모듈은 부모 모듈의 `locals` 값을 `local.이름`으로 직접 참조할 수 있다.

**33.** 누군가 콘솔에서 VM 태그를 손으로 바꿨고, 코드에는 예전 태그 값이 그대로 있다. 다음 `terraform plan`의 결과로 가장 알맞은 것은?
- A. 아무 변화가 없다고 나온다
- B. 태그를 코드의 값으로 되돌리는 변경을 제안한다
- C. VM을 반드시 교체하겠다고 나온다
- D. 코드가 콘솔 값으로 자동 수정된다

**34.** (참과 거짓) `terraform validate`는 `terraform init`을 한 번도 하지 않은 디렉터리에서도 provider 리소스의 인수까지 검사할 수 있다.

**35.** HCP Terraform 워크스페이스의 실행 모드를 `Local`로 바꾸면 HCP Terraform이 맡는 일은?
- A. plan과 apply를 모두 원격에서 실행한다
- B. state를 저장(하고 잠금)하며, 실행은 내 PC에서 한다
- C. 정책 검사만 한다
- D. 아무 역할도 하지 않는다

**36.** `depends_on`에 대한 설명으로 옳은 것은? **(두 개 선택)**
- A. 참조로 드러나지 않는 숨은 의존성을 표현할 때 쓴다
- B. resource 블록과 module 블록에 쓸 수 있다
- C. 모든 리소스에 습관적으로 쓰는 것이 권장된다
- D. 리소스 생성에 걸릴 시간을 지정한다

**37.** 이 설정을 Terraform 1.12 이상에서만 실행되게 하려면?
- A. `provider "terraform" { version = ">= 1.12" }`
- B. `terraform { required_version = ">= 1.12" }`
- C. 작업 디렉터리에 `.terraform-version` 파일을 둔다
- D. `required_providers { terraform = ">= 1.12" }`

**38.** 원격 backend의 state를 가공 없이 그대로 내려받아 표준 출력으로 보내는 명령은?
- A. `terraform state list`
- B. `terraform state pull`
- C. `terraform state push`
- D. `terraform output -json`

**39.** `version` 인수를 쓸 수 **없는** 모듈 소스는? **(두 개 선택)**
- A. `"./modules/app"`
- B. `"git::https://example.com/app.git?ref=v2.1.0"`
- C. `"acme/app/aws"`
- D. `"app.terraform.io/acme/app/aws"`

**40.** 잠금 파일이 있는 상태에서, 버전 제약 범위 안의 최신 provider로 올리고 잠금 파일도 갱신하는 명령은?
- A. `terraform init`
- B. `terraform init -upgrade`
- C. `terraform providers lock`
- D. `terraform get -update`

---

## 4. 문항 41~57

**41.** `set(string)` 타입의 특징은? **(두 개 선택)**
- A. 중복 원소가 없다
- B. 원소의 순서가 보장된다
- C. `[0]` 같은 인덱스로 원소에 접근할 수 없다
- D. 원소마다 다른 타입을 섞을 수 있다

**42.** state에 있는 리소스 **하나의 모든 속성**을 확인하는 명령은?
- A. `terraform state list`
- B. `terraform state show ADDR`
- C. `terraform output`
- D. `terraform providers`

**43.** (참과 거짓) HCP Terraform에서 팀 권한은 워크스페이스 단위로만 줄 수 있고 프로젝트 단위로는 줄 수 없다.

**44.** (참과 거짓) Vault provider의 `data` 소스로 읽은 비밀 값은 state에 기록되지 않는다.

**45.** Terraform의 주된 용도로 가장 알맞은 것은?
- A. 서버 OS 안의 패키지와 설정 파일 관리 전용 도구
- B. 클라우드·온프레미스·SaaS 인프라를 코드로 프로비저닝하고 관리하는 도구
- C. 컨테이너 이미지 빌드 도구
- D. 애플리케이션 로그 수집 도구

**46.** `count`로 만든 `local_file.page[0]`을 `for_each` 키 주소 `local_file.page["home"]`으로 바꾸면서, 파일을 다시 만들지 않고 plan으로 확인하며 옮기려 한다. 알맞은 방법은?
- A. `terraform state rm` 후 apply
- B. `terraform apply -replace`
- C. `removed` 블록
- D. `moved` 블록

**47.** `terraform apply`가 의존 관계 없는 리소스를 동시에 처리하는 기본 개수는?
- A. 1
- B. 5
- C. 10
- D. 제한 없음

**48.** 리소스가 만들어진 **뒤** 그 리소스의 속성이 조건을 만족하는지 검사하고, 실패하면 오류로 멈추게 하려면?
- A. 변수 `validation`
- B. `precondition`
- C. `postcondition`
- D. `check` 블록

**49.** HCP Terraform 비공개 레지스트리에 모듈을 게시하는 일반적인 방법은?
- A. VCS 저장소를 연결하고 버전 태그를 붙인다
- B. `terraform.tfvars`를 업로드한다
- C. `terraform push` 명령을 쓴다
- D. 비공개 레지스트리에는 provider만 올릴 수 있다

**50.** 레지스트리에서 HashiCorp가 직접 소유하고 관리하는 provider의 등급은?
- A. Official
- B. Partner
- C. Community
- D. Archived

**51.** 공개 Terraform 레지스트리에 모듈을 게시하려 할 때 GitHub 저장소 이름 형식은?
- A. `<NAME>-terraform-module`
- B. `tf-module-<NAME>`
- C. 형식 제한이 없다
- D. `terraform-<PROVIDER>-<NAME>`

**52.** HashiCorp 문서가 설명하는 Terraform 핵심 워크플로의 세 단계는?
- A. Init, Plan, Destroy
- B. Code, Build, Deploy
- C. Plan, Apply, Refresh
- D. Write, Plan, Apply

**53.** `terraform workspace new`를 한 번도 실행하지 않은 디렉터리에서 `terraform.workspace`의 값은?
- A. `"main"`
- B. `"default"`
- C. `"dev"`
- D. 빈 문자열

**54.** 폐기 예정인 `terraform refresh` 대신 권장되는 방법은?
- A. `terraform apply -refresh-only`
- B. `terraform plan -refresh=false`
- C. `terraform state push`
- D. `terraform init -reconfigure`

**55.** (참과 거짓) `alias`로 같은 provider의 설정을 두 개 만들면 provider 플러그인도 두 번 설치된다.

**56.** 로컬 state로 운영하던 설정을 HCP Terraform 워크스페이스로 옮기는 기본 절차는?
- A. `terraform state push`로 app.terraform.io에 직접 업로드한다
- B. `terraform login`만 하면 자동으로 옮겨진다
- C. `cloud` 블록을 추가하고 `terraform init`을 실행해 마이그레이션한다
- D. 옮길 수 없으므로 리소스를 새로 만든다

**57.** `var.amis`에 `"seoul"` 키가 없을 때 `lookup(var.amis, "seoul", "ami-default")`의 결과는?
- A. 오류
- B. `null`
- C. `"ami-default"`
- D. `""`

---

## 5. 정답표

| 번호 | 정답 | 목표 | 번호 | 정답 | 목표 | 번호 | 정답 | 목표 |
|---|---|---|---|---|---|---|---|---|
| 1 | C | 1 | 20 | C | 7 | 39 | A, B | 5 |
| 2 | 참 | 3 | 21 | B | 5 | 40 | B | 3 |
| 3 | A, C | 2 | 22 | 거짓 | 2 | 41 | A, C | 4 |
| 4 | C | 4 | 23 | C | 8 | 42 | B | 7 |
| 5 | A | 6 | 24 | B | 4 | 43 | 거짓 | 8 |
| 6 | A | 3 | 25 | B | 6 | 44 | 거짓 | 4 |
| 7 | C | 8 | 26 | A | 3 | 45 | B | 1 |
| 8 | 거짓 | 4 | 27 | B | 4 | 46 | D | 6 |
| 9 | C | 7 | 28 | A, B | 7 | 47 | C | 3 |
| 10 | B | 5 | 29 | 참 | 1 | 48 | C | 4 |
| 11 | A, B | 1 | 30 | B | 2 | 49 | A | 8 |
| 12 | 거짓 | 6 | 31 | B | 4 | 50 | A | 2 |
| 13 | C | 2 | 32 | 거짓 | 5 | 51 | D | 5 |
| 14 | C | 3 | 33 | B | 6 | 52 | D | 3 |
| 15 | A, B | 4 | 34 | 거짓 | 3 | 53 | B | 6 |
| 16 | A, B, C | 8 | 35 | B | 8 | 54 | A | 7 |
| 17 | A, B, D | 6 | 36 | A, B | 4 | 55 | 거짓 | 2 |
| 18 | B | 3 | 37 | B | 2 | 56 | C | 8 |
| 19 | B | 4 | 38 | B | 6 | 57 | C | 4 |

## 6. 목표별 채점표

| 목표 | 문항 번호 | 문항 수 | 내 오답 수 | 다시 볼 편 |
|---|---|---|---|---|
| 1. IaC with Terraform | 1, 11, 29, 45 | 4 | | [T01](./t01-iac-workflow-providers.md) 1장 |
| 2. Terraform fundamentals | 3, 13, 22, 30, 37, 50, 55 | 7 | | [T01](./t01-iac-workflow-providers.md) 2장, [T03](./t03-state-backend-import.md) 1장 |
| 3. Core workflow | 2, 6, 14, 18, 26, 34, 40, 47, 52 | 9 | | [T01](./t01-iac-workflow-providers.md) 3장 |
| 4. Configuration | 4, 8, 15, 19, 24, 27, 31, 36, 41, 44, 48, 57 | 12 | | [T02](./t02-configuration-modules.md) 1~3장 |
| 5. Modules | 10, 21, 32, 39, 51 | 5 | | [T02](./t02-configuration-modules.md) 4장 |
| 6. State management | 5, 12, 17, 25, 33, 38, 46, 53 | 8 | | [T03](./t03-state-backend-import.md) 1~3장 |
| 7. Maintain infrastructure | 9, 20, 28, 42, 54 | 5 | | [T03](./t03-state-backend-import.md) 3장 |
| 8. HCP Terraform | 7, 16, 23, 35, 43, 49, 56 | 7 | | [T04](./t04-hcp-terraform.md) |
| **합계** | | **57** | | 40점(70%) 이상이면 합격 참고선 통과 |

**결과 해석**

- **48점 이상:** 실제 시험 준비가 거의 끝났습니다. 공식 [샘플 문항](https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-questions-004)과 [리뷰 가이드](https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-review-004)를 훑고 응시하세요.
- **40~47점:** 합격선 근처입니다. 오답이 2개 이상인 목표의 편을 다시 읽고, 해당 편의 창작 문항을 다시 풉니다.
- **40점 미만:** 오답이 몰린 편의 **실습을 다시** 합니다. 특히 목표 3·4·6은 명령을 직접 쳐 봐야 오래 남습니다.
- 목표별로 **절반 이상 틀린 목표가 하나라도 있으면** 총점과 상관없이 그 편부터 다시 보세요. 실제 시험의 목표별 비중은 공개되지 않았습니다.

---

## 7. 해설

<details><summary>1~10</summary>

**1. C** — 선언형은 "무엇을"(최종 상태)을 적고, 순서와 작업은 Terraform이 의존성 그래프로 계산합니다. A는 명령형 설명입니다. 근거: [What is Terraform?](https://developer.hashicorp.com/terraform/intro)

**2. 참** — plan은 state와 실제 인프라를 읽어 비교만 합니다. 변경은 apply에서 일어납니다. 근거: [terraform plan](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/plan)

**3. A, C** — provider는 `init`이 설치하는 플러그인이고, 한 설정에 여러 개를 쓸 수 있습니다. 버전은 `required_providers`에 쓰며(provider 블록의 `version`은 폐기 예정), provider는 Terraform과 따로 릴리스됩니다. 근거: [Providers](https://developer.hashicorp.com/terraform/language/v1.12.x/providers), [Provider requirements](https://developer.hashicorp.com/terraform/language/v1.12.x/providers/requirements)

**4. C** — 명령줄의 `-var-file`이 `terraform.tfvars`와 `TF_VAR_`보다 우선합니다. 여러 곳에 있다고 오류가 나지는 않습니다. 근거: [Input variables](https://developer.hashicorp.com/terraform/language/v1.12.x/values/variables)

**5. A** — local backend는 `path` 인수로 state 파일 경로를 지정합니다. B·C의 옵션과 변수는 없습니다. 근거: [Backend type: local](https://developer.hashicorp.com/terraform/language/backend/local)

**6. A** — 새 provider는 `init`으로 설치하고 잠금 파일에 기록해야 합니다. `terraform get`은 모듈만 받습니다. 근거: [terraform init](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/init)

**7. C** — VCS 기반 워크스페이스는 연결된 브랜치에 커밋이 들어오면 run을 시작하고, PR에는 speculative plan을 돌립니다. 근거: [UI and VCS-driven run workflow](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/run/ui)

**8. 거짓** — 한 블록에 `count`와 `for_each`를 동시에 쓸 수 없습니다. 근거: [The for_each meta-argument](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each)

**9. C** — `import` 블록은 Terraform 1.5에서 도입됐고, `for_each`를 함께 쓰는 대량 import는 1.7부터입니다. 근거: [Import](https://developer.hashicorp.com/terraform/language/v1.12.x/import)

**10. B** — 모듈 출력은 `module.<모듈 이름>.<출력 이름>`으로 참조합니다. 근거: [Modules — accessing module output values](https://developer.hashicorp.com/terraform/language/v1.12.x/modules/configuration)
</details>

<details><summary>11~20</summary>

**11. A, B** — 같은 언어·워크플로로 여러 provider를 다루고, 리소스 참조로 provider 사이 의존성을 표현합니다. 리소스 타입은 provider마다 다르고(C), 비용 최적화는 자동이 아닙니다(D). 근거: [Use cases](https://developer.hashicorp.com/terraform/intro/v1.12.x/use-cases)

**12. 거짓** — 잠금을 지원하는 backend라면 state를 쓸 수 있는 작업에서 자동으로 잠급니다. `terraform lock`이라는 명령은 없습니다. 근거: [State locking](https://developer.hashicorp.com/terraform/language/state/locking)

**13. C** — 잠금 파일은 provider의 버전·제약·해시만 기록합니다. 모듈 버전은 기록하지 않습니다. 근거: [Dependency lock file](https://developer.hashicorp.com/terraform/language/v1.12.x/files/dependency-lock)

**14. C** — `fmt`는 기본적으로 현재 디렉터리만 처리하며, `-recursive`로 하위 디렉터리까지 포함합니다. 근거: [terraform fmt](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/fmt)

**15. A, B** — `sensitive`는 화면 출력만 가립니다. state와 저장된 plan 파일에는 값이 남으므로 둘 다 보호해야 합니다. 근거: [Manage sensitive data](https://developer.hashicorp.com/terraform/language/v1.12.x/manage-sensitive-data)

**16. A, B, C** — 변수 세트는 전역, 프로젝트, 선택한 워크스페이스 범위로 적용합니다. 한 run에만 쓰는 값은 run 변수로 넣는 것이지 변수 세트의 범위가 아닙니다. 근거: [Variables](https://developer.hashicorp.com/terraform/cloud-docs/variables)

**17. A, B, D** — 원격 backend는 공유, 잠금(지원 backend), 저장소 수준의 암호화·접근 제어를 줍니다. 민감 값이 state에서 사라지지는 않습니다. 근거: [Backend configuration](https://developer.hashicorp.com/terraform/language/v1.12.x/backend)

**18. B** — 저장된 plan 파일을 apply하면 그 계획만 실행되고, 그사이 state가 바뀌면 오래된 계획이라며 거부합니다. 근거: [terraform apply](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/apply)

**19. B** — data 소스는 읽기 전용이므로 destroy 대상이 아닙니다. 근거: [Data sources](https://developer.hashicorp.com/terraform/language/v1.12.x/data-sources)

**20. C** — `TRACE`가 가장 자세합니다. 그 아래로 `DEBUG`, `INFO`, `WARN`, `ERROR`가 있고, `JSON`은 TRACE 수준을 JSON 형식으로 냅니다. 근거: [Debugging Terraform](https://developer.hashicorp.com/terraform/internals/v1.12.x/debugging)
</details>

<details><summary>21~30</summary>

**21. B** — `~> 1.0`은 `>= 1.0, < 2.0`입니다. `>= 1.0`(A)은 2.x도 허용하고, `~> 1.0.0`(C)은 1.0.x만 허용합니다. D는 지원하지 않는 형식입니다. 근거: [Version constraints](https://developer.hashicorp.com/terraform/language/expressions/version-constraints)

**22. 거짓** — 대부분의 API는 "누가 만들었는지"를 알려 주지 않으므로, Terraform은 state로 설정과 실제 객체를 연결합니다. 근거: [Purpose of Terraform state](https://developer.hashicorp.com/terraform/language/v1.12.x/state/purpose)

**23. C** — OPA 정책은 Rego로, Sentinel 정책은 Sentinel 언어로 씁니다. 근거: [Policy enforcement](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/policy-enforcement)

**24. B** — `if` 절이 `"b"`를 걸러 내고 나머지를 대문자로 바꿉니다. 대괄호이므로 결과는 리스트입니다. 근거: [for expressions](https://developer.hashicorp.com/terraform/language/expressions/for)

**25. B** — `-reconfigure`는 기존 state를 옮기지 않고 backend 설정을 새로 합니다. 옮기려면 `-migrate-state`입니다. 근거: [terraform init](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/init)

**26. A** — `terraform show`에 plan 파일을 주면 사람이 읽는 형식으로, `-json`을 붙이면 JSON으로 보여 줍니다. plan 파일은 바이너리입니다. 근거: [terraform show](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/show)

**27. B** — dynamic 블록의 반복 변수 이름은 기본적으로 **블록 이름**(`setting`)이며 `.key`, `.value`를 가집니다. `iterator` 인수로 이름을 바꿀 수 있습니다. 근거: [Dynamic blocks](https://developer.hashicorp.com/terraform/language/expressions/dynamic-blocks)

**28. A, B** — import 블록은 plan에서 미리 보고, 여러 개를 한 번에, 코드 초안 생성까지 지원합니다. 어느 방식이든 provider가 import를 지원해야 하며 결과는 state에 기록됩니다. 근거: [Import](https://developer.hashicorp.com/terraform/language/v1.12.x/import), [Generating configuration](https://developer.hashicorp.com/terraform/language/import/generating-configuration)

**29. 참** — 인프라 정의가 코드이므로 Git의 리뷰·이력·되돌리기를 그대로 씁니다. 근거: [Infrastructure as code](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code)

**30. B** — 모듈 블록의 `providers` 맵으로 "자식의 이름 = 부모의 설정"을 넘깁니다. 모듈 블록에는 단수형 `provider` 인수가 없습니다. 근거: [Providers within modules](https://developer.hashicorp.com/terraform/language/v1.12.x/modules/develop/providers)
</details>

<details><summary>31~40</summary>

**31. B** — `prevent_destroy`는 그 리소스를 지우는 plan을 오류로 막습니다. 다만 resource 블록 자체를 코드에서 지우면 보호 설정도 사라집니다. 근거: [The lifecycle meta-argument](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)

**32. 거짓** — 자식 모듈은 부모의 변수·local·리소스를 직접 볼 수 없습니다. 입력 변수로 받아야 합니다. 근거: [Modules — module block](https://developer.hashicorp.com/terraform/language/v1.12.x/modules/configuration)

**33. B** — plan은 refresh로 drift를 발견하고 코드(원하는 상태)로 되돌리는 계획을 세웁니다. 태그는 보통 그 자리에서 수정 가능한 속성입니다. 근거: [Manage resource drift](https://developer.hashicorp.com/terraform/tutorials/state/resource-drift)

**34. 거짓** — validate는 provider 스키마가 필요하므로 초기화된 디렉터리에서 실행합니다. 근거: [terraform validate](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/validate)

**35. B** — Local 실행 모드에서는 plan·apply가 내 PC에서 돌고, HCP Terraform은 state 저장소 역할을 합니다. 근거: [Remote operations](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/run/remote-operations)

**36. A, B** — `depends_on`은 숨은 의존성에만, 리소스와 모듈 블록에 씁니다. 남용하면 plan이 보수적으로 바뀝니다. 근거: [The depends_on meta-argument](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on)

**37. B** — Terraform CLI 버전 제약은 `terraform` 블록의 `required_version`입니다. 근거: [terraform block](https://developer.hashicorp.com/terraform/language/v1.12.x/block/terraform)

**38. B** — `state pull`은 현재 backend의 state를 그대로 표준 출력에 씁니다. `push`는 반대 방향이며 위험합니다. 근거: [terraform state pull](https://developer.hashicorp.com/terraform/cli/commands/state/pull)

**39. A, B** — `version`은 공개·비공개 레지스트리 소스에만 씁니다. Git은 `ref`로 고정합니다. 근거: [Module sources](https://developer.hashicorp.com/terraform/language/modules/sources)

**40. B** — `init -upgrade`는 잠금 파일을 무시하고 제약 범위 안의 최신 버전을 다시 골라 잠금 파일을 갱신합니다. `providers lock`은 해시를 추가하는 용도입니다. 근거: [Dependency lock file](https://developer.hashicorp.com/terraform/language/v1.12.x/files/dependency-lock)
</details>

<details><summary>41~50</summary>

**41. A, C** — set은 중복이 없고 순서가 없어 인덱스로 접근할 수 없습니다. 컬렉션 타입이라 원소 타입이 모두 같습니다. 근거: [Type constraints](https://developer.hashicorp.com/terraform/language/v1.12.x/expressions/type-constraints)

**42. B** — `state show ADDR`는 리소스 하나의 속성을, `state list`는 주소 목록을 보여 줍니다. 근거: [terraform state list](https://developer.hashicorp.com/terraform/cli/v1.12.x/commands/state/list), [terraform state show](https://developer.hashicorp.com/terraform/cli/commands/state/show)

**43. 거짓** — 팀 권한은 조직, 프로젝트, 워크스페이스 단위로 줄 수 있습니다(팀 기능은 유료 에디션). 근거: [Permissions](https://developer.hashicorp.com/terraform/cloud-docs/users-teams-organizations/permissions)

**44. 거짓** — data 소스로 읽은 값은 state에 기록됩니다. state에 남기지 않으려면 ephemeral 리소스(1.10+)와 write-only 인수(1.11+)를 검토합니다. 근거: [Manage sensitive data](https://developer.hashicorp.com/terraform/language/v1.12.x/manage-sensitive-data)

**45. B** — Terraform은 API가 있는 인프라·서비스를 코드로 프로비저닝·관리하는 도구입니다. OS 안 설정 관리는 Ansible 같은 구성 관리 도구의 주 영역입니다. 근거: [What is Terraform?](https://developer.hashicorp.com/terraform/intro)

**46. D** — `moved { from = local_file.page[0]  to = local_file.page["home"] }`로 인스턴스 주소를 옮기면 재생성 없이 plan에 "has moved to"로 표시됩니다. 근거: [Refactoring](https://developer.hashicorp.com/terraform/language/v1.12.x/state/refactor)

**47. C** — 기본 동시 작업 수는 10이며 `-parallelism=N`으로 바꿉니다. 근거: [Resource graph](https://developer.hashicorp.com/terraform/internals/v1.12.x/graph)

**48. C** — postcondition은 리소스를 만들거나 읽은 뒤 검사하고, 실패하면 오류입니다. check 블록은 경고만 냅니다. 근거: [Validate your configuration](https://developer.hashicorp.com/terraform/language/v1.12.x/validate)

**49. A** — 비공개 레지스트리 모듈은 보통 VCS 저장소를 연결하고 버전 태그로 게시합니다. 근거: [Publish private modules](https://developer.hashicorp.com/terraform/cloud-docs/registry/publish-modules)

**50. A** — Official은 HashiCorp가, Partner는 검증된 외부 회사가, Community는 개인·커뮤니티가 관리합니다. 근거: [Providers in the registry](https://developer.hashicorp.com/terraform/registry/providers)
</details>

<details><summary>51~57</summary>

**51. D** — 공개 레지스트리는 `terraform-<PROVIDER>-<NAME>` 형식의 공개 GitHub 저장소를 요구합니다. 근거: [Publishing modules](https://developer.hashicorp.com/terraform/registry/modules/publish)

**52. D** — 핵심 워크플로는 Write → Plan → Apply입니다. 근거: [The core Terraform workflow](https://developer.hashicorp.com/terraform/intro/v1.12.x/core-workflow)

**53. B** — 처음에는 `default` 워크스페이스 하나가 있으며 삭제할 수 없습니다. 근거: [Workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces)

**54. A** — `apply -refresh-only`는 외부 변경을 보여 주고 승인받은 뒤 state를 갱신합니다. `refresh`는 확인 없이 바로 반영해 위험합니다. 근거: [Use refresh-only mode to sync Terraform state](https://developer.hashicorp.com/terraform/tutorials/state/refresh)

**55. 거짓** — 별칭은 같은 provider의 **설정**을 하나 더 만드는 것이며 플러그인은 한 번만 설치됩니다. 근거: [Provider configuration](https://developer.hashicorp.com/terraform/language/providers/configuration)

**56. C** — `cloud` 블록을 추가하고 `terraform init`을 실행하면 기존 state를 HCP Terraform 워크스페이스로 옮길지 안내합니다(먼저 `terraform login` 필요). 근거: [Migrate to HCP Terraform](https://developer.hashicorp.com/terraform/cli/cloud/migrating)

**57. C** — `lookup(map, key, default)`는 키가 없으면 기본값을 돌려줍니다. 근거: [lookup function](https://developer.hashicorp.com/terraform/language/functions/lookup)
</details>

---

## 8. 복기 방법

1. 틀린 문항마다 해설의 **근거 문서**를 열어 해당 단락을 읽습니다.
2. 명령 문항(2·6·14·18·25·26·38·40·54)을 틀렸다면 T01·T03 실습 디렉터리에서 **그 명령을 직접 실행**해 결과를 봅니다.
3. "코드·state·실제 인프라 중 무엇이 바뀌나"를 묻는 문항(12·17·33·38·46·54)은 T03 그림을 안 보고 다시 그려 봅니다.
4. 일주일 뒤 **문항 순서를 섞어** 다시 풉니다. 답의 위치를 외운 것인지, 이해한 것인지 구분됩니다.

---

## ⏭️ 다음 단계

모의고사를 40점 이상으로 통과했다면 응시 준비는 마무리 단계입니다.

**응시 전 체크리스트**

- [ ] [자격 페이지](https://developer.hashicorp.com/certifications/terraform-associate)에서 시험 버전(004)·기준 Terraform 버전(1.12)·응시료·형식을 **다시 확인**
- [ ] 공식 [학습 가이드](https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-study-004)의 목표별 링크를 훑고, [샘플 문항](https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-questions-004)으로 형식 확인
- [ ] 온라인 감독 시험 규정(신분증, 응시 환경, 금지 물품)을 자격 페이지의 "Exam rules and requirements" 링크로 확인
- [ ] T05를 문항 순서를 섞어 한 번 더, 목표별 오답 0~1개

**합격 뒤 이어 갈 길**

| 방향 | 이 시리즈에서 | 메모 |
|---|---|---|
| 실제 클라우드에 적용 | 클라우드 탭 [cloud05](../../cloud/cloud05-terraform-aws.md) — Terraform으로 AWS 인프라(모듈, S3 원격 state + 잠금) | 💰 비용 발생. 실습 후 `terraform destroy` 체크리스트 |
| 홈랩에 다시 적용 | [Day 35](../../05-capstone/day35-terraform.md)의 `k8s-openstack-tf`에 T02의 `validation`·`for_each`, T03의 원격 backend·`moved` 적용 | 비용 없음 |
| IaC 보안 | [해킹 24](../../hacking/hack24-iac-scanning.md) — Checkov·Trivy로 Terraform 점검, T04의 정책 검사·run task와 역할 나누기 | — |
| 다음 HashiCorp 자격 | **Terraform Infrastructure Engineer Professional**(객관식, Terraform 1.15 기준), **Terraform Authoring & Operations Advanced**(실기 랩), **Vault Associate**(T02의 Vault 비밀 관리와 연결) | 2026-11-19 [자격 목록](https://developer.hashicorp.com/certifications/infrastructure-automation) 기준. 응시 전 공식 페이지 재확인 |

이 탭의 모든 문항은 공식 목표를 바탕으로 직접 만든 연습 문제였습니다. 덤프로 외운 답이 아니라 **명령을 쳐 보고 문서로 확인한 이해**가 시험장과 실무 모두에서 통합니다.
