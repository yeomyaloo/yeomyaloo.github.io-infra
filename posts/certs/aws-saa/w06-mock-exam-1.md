---
title: "AWS SAA W06. 모의고사 1 — 기본 난이도 (65문항 · 130분)"
date: 2026-11-19T22:35:17+09:00
weight: 7
tags: ["aws-saa", "certification", "aws", "mock-exam"]
summary: "[직접 만든 창작 문항 · 기출·덤프 아님] 실제 시험과 같은 65문항을 130분 안에 한 번에 풀어 본다. 시나리오에서 요구 사항(가용성·보안·성능·비용)과 꼬리표(\"가장 비용 효율적인\", \"운영 부담이 가장 적은\") 를 먼저 찾는 습관을 만든다."
image: ""
draft: false
---
> **자격증 준비 · AWS SAA** | 난이도 ★★★☆☆ | 예상 소요 시험 130분 + 채점·복기 1.5~2시간

> ⚠️ **이 모의고사의 모든 문항은 AWS가 공개한 SAA-C03 시험 가이드(2026-11-19 확인, 가이드 버전 1.1)의 4개 도메인과 과제 설명을 바탕으로 이 시리즈가 직접 창작한 연습 문제입니다.** 실제 시험 문제나 유출 문제(덤프)가 아니고, 그런 것을 재현하지도 않습니다. AWS 자격증 시험은 응시 계약(NDA)으로 보호되므로 "실제 출제 문제"라고 주장하는 자료는 믿지도, 퍼뜨리지도 마세요. 회사 이름·수치·시나리오는 모두 새로 지은 것입니다.

## 🎯 오늘의 목표

- 실제 시험과 같은 **65문항을 130분 안에** 한 번에 풀어 본다
- 시나리오에서 **요구 사항(가용성·보안·성능·비용)과 꼬리표("가장 비용 효율적인", "운영 부담이 가장 적은")** 를 먼저 찾는 습관을 만든다
- 정답표와 해설로 틀린 이유를 확인하고, **도메인별 점수표**로 약한 도메인을 찾아 W01~W05로 돌아갈 곳을 정한다

---

## 1. 시험 안내

### 1-1. 형식과 규칙

| 항목 | 이 모의고사 | 실제 시험(2026-11-19 AWS 공식 페이지·시험 가이드 확인, 자세한 내용은 [W00](./w00-exam-overview.md)) |
|---|---|---|
| 시험 코드 | SAA-C03 기준 | **SAA-C03** (시험 가이드 개정 이력: 1.0 2026-08-28, 1.1 2026-10-29) |
| 문항 수 | **65문항**, 모두 채점 | **65문항** = 채점 50문항 + 채점하지 않는 15문항(어느 문항인지 표시되지 않음) |
| 시간 | **130분** | **130분** |
| 문항 형식 | 단일 선택(보기 4개) + 복수 선택(보기 5개 중 2개) | 단일 선택(정답 1, 오답 3) + 복수 선택(보기 5개 이상 중 정답 2개 이상) |
| 채점 | 맞힌 개수 → 도메인별 정답률 | **100~1,000 척도 점수, 합격 720점**. 도메인별로 따로 통과할 필요 없는 **보상(compensatory) 모델** |
| 오답 감점 | 없음 | 없음(빈칸은 오답). **모르면 반드시 찍기** |
| 합격 참고선 | **47문항(72%)**, 안전선 52문항(80%) | 척도 점수는 문항 난이도로 환산되므로 정답률과 1:1로 대응하지 않음 |
| 도메인 비중 | 공식 비중에 맞춤(아래 2절) | D1 보안 30% · D2 복원력 26% · D3 고성능 24% · D4 비용 20% |
| 응시 | 종이·메모장 | Pearson VUE 시험 센터 또는 온라인 감독, 응시료 150 USD, **한국어 시험 제공** |

> 💡 72%는 "720/1,000"을 정답률로 단순 환산한 **참고값**일 뿐 AWS가 공표한 기준이 아닙니다. 실제 점수는 문항 난이도에 따라 환산되므로, 모의고사에서는 **80% 이상을 안정적으로** 내는 것을 목표로 잡으세요.

### 1-2. 진행 방법

1. 타이머를 **130분**으로 맞추고 시작합니다. 중간에 해설을 펼치지 마세요(정답표와 해설은 문항이 다 끝난 뒤에 있습니다).
2. 답은 종이나 메모장에 `Q1 B`, `Q2 A,D`처럼 적습니다. 헷갈린 문항은 번호 옆에 `?`를 붙여 **검토 표시**를 합니다(실제 시험의 "Flag for review"와 같은 역할).
3. 복수 선택 문항은 문제에 "(2개 선택)"이 적혀 있습니다. 실제 시험처럼 **모두 맞아야** 정답입니다.
4. 시간 배분 권장: 1차로 **100분 안에** 전부 풀고(문항당 약 1분 30초), 남은 30분에 `?` 문항만 다시 봅니다.
5. 끝나면 5절 정답표로 채점하고, 7절 도메인별 점수표를 채웁니다.

### 1-3. 문항을 읽는 순서

| 순서 | 할 일 | 예 |
|---|---|---|
| ① | **마지막 문장(질문)** 부터 읽는다 | "가장 비용 효율적인 방법은?" → 비용이 판정 기준 |
| ② | 시나리오에서 **조건 단어**에 밑줄 | "사람 개입 없이", "인터넷 경로 금지", "순서대로", "UDP" |
| ③ | 보기마다 **조건을 어기는 것**부터 지운다 | "퍼블릭 서브넷으로 이동" → 보안 조건 위반 |
| ④ | 남은 둘 중 **꼬리표**로 고른다 | 둘 다 되면 관리형·추가 코드 없는 쪽 |

---

## 2. 문항 구성

| 도메인 | 공식 비중 | 이 모의고사 문항 수 |
|---|---|---|
| D1 안전한 아키텍처 설계 | 30% | 20 |
| D2 복원력 있는 아키텍처 설계 | 26% | 17 |
| D3 고성능 아키텍처 설계 | 24% | 15 |
| D4 비용 최적화 아키텍처 설계 | 20% | 13 |
| **합계** | 100% | **65** |

실제 시험처럼 도메인을 섞어 배치했습니다. 어느 문항이 어느 도메인인지는 5절 정답표에서 확인합니다. 복수 선택 문항은 4개입니다.

---

## 3. 문항

> ⏱️ **지금 타이머를 시작하세요.** 130분.

**Q1.** SaaS 회사는 EC2와 Fargate를 합쳐 시간당 평균 40달러를 1년 내내 꾸준히 쓴다. 내년에 일부 워크로드를 Lambda로 옮기고 인스턴스 패밀리도 바꿀 예정이다. 가장 비용 효율적인 방법은?

- A. EC2 Instance Savings Plans를 구매한다
- B. 시간당 약 40달러 이하의 1년 Compute Savings Plans를 구매한다
- C. 모든 워크로드를 Spot으로 바꾼다
- D. 현재 인스턴스에 맞춘 3년 Standard RI를 구매한다

**Q2.** 회사 웹 애플리케이션은 단일 AZ의 EC2 두 대와 그 앞의 ALB로 운영된다. AZ 하나가 장애 나도 서비스가 계속되어야 하고, 인스턴스가 고장 나면 자동으로 교체되어야 한다. 알맞은 구성은?

- A. 같은 AZ에 인스턴스를 네 대로 늘린다
- B. 인스턴스를 더 큰 유형으로 바꾼다
- C. 각 인스턴스에 탄력적 IP를 붙이고 Route 53 단순 라우팅을 쓴다
- D. 두 개 이상 AZ의 서브넷에 걸친 Auto Scaling 그룹을 만들고 ALB 대상 그룹에 연결하며 ELB 상태 확인을 사용한다

**Q3.** IoT 수집 서비스는 DynamoDB 테이블에 하루 종일 **거의 일정한** 쓰기 트래픽을 보낸다. 지금은 온디맨드 모드다. 성능을 유지하면서 비용을 줄이는 방법은?

- A. 테이블을 Standard-IA 테이블 클래스로 바꾼다
- B. 글로벌 테이블로 바꾼다
- C. 프로비저닝 용량 모드로 바꾸고 Auto Scaling을 켠다(필요하면 예약 용량 구매)
- D. 온디맨드 모드를 유지하고 DAX를 추가한다

**Q4.** 회사 VPC의 애플리케이션이 다른 AWS 계정에 있는 결제 SaaS 공급자의 API를 호출해야 한다. 트래픽은 인터넷을 지나면 안 되고, 두 VPC의 CIDR이 **겹친다**. 공급자는 NLB 뒤에서 서비스를 운영한다. 알맞은 방법은?

- A. VPC 피어링을 연결한다
- B. 공급자 API를 퍼블릭으로 열고 보안 그룹으로 회사 IP만 허용한다
- C. 공급자가 NLB로 VPC 엔드포인트 서비스(PrivateLink)를 만들고, 회사 VPC에 인터페이스 엔드포인트를 만든다
- D. Transit Gateway에 두 VPC를 연결한다

**Q5.** 회사는 20개 계정에서 EC2·EBS·RDS·DynamoDB·EFS를 쓴다. 모든 계정에 **같은 백업 정책**(매일 백업, 35일 보존, 다른 리전 복사)을 강제하고 중앙에서 준수 여부를 보고 싶다. 알맞은 방법은?

- A. 각 계정에서 서비스별로 스냅숏 스크립트를 cron으로 돌린다
- B. Data Lifecycle Manager만 사용한다
- C. AWS Organizations에서 AWS Backup 백업 정책을 만들어 OU에 연결하고, Backup Audit Manager로 준수 여부를 본다
- D. S3 복제로 모든 데이터를 한 버킷에 모은다

**Q6.** ALB로 HTTPS를 서비스하려 한다. 인증서 발급 비용과 **갱신 작업을 없애고** 싶다. 알맞은 방법은?

- A. AWS Certificate Manager(ACM)에서 퍼블릭 인증서를 DNS 검증으로 발급해 ALB 리스너에 연결한다
- B. 자체 서명 인증서를 만들어 ALB에 올린다
- C. CloudHSM에 인증서를 저장한다
- D. EC2에서 Let's Encrypt 클라이언트를 돌려 매달 IAM에 인증서를 올린다

**Q7.** 증권 주문 시스템은 계좌별로 주문이 **들어온 순서대로**, 그리고 **중복 없이** 처리되어야 한다. 계좌는 수십만 개다. 알맞은 구성은?

- A. SNS 표준 토픽으로 처리 서버에 보낸다
- B. SQS FIFO 큐에 계좌 ID를 메시지 그룹 ID로, 주문 ID를 중복 제거 ID로 넣는다
- C. SQS FIFO 큐에 모든 메시지를 같은 그룹 ID로 넣는다
- D. SQS 표준 큐에 넣고 타임스탬프로 정렬한다

**Q8.** 티켓 예매 이벤트 때 주문 API가 초당 수만 건을 받는데, 뒤쪽 주문 처리 서버는 초당 수천 건만 처리한다. 주문을 **잃지 않고** 순차 처리하면 되며 처리 지연은 몇 분까지 괜찮다. 가장 알맞은 구조는?

- A. 처리 서버 수를 이벤트 최대치에 맞춰 미리 늘려 둔다
- B. API가 주문을 SQS 큐에 넣고, 처리 서버(Auto Scaling 그룹)가 큐에서 꺼내 처리하며 큐 길이 기준으로 확장한다
- C. API와 처리 서버 사이에 NLB를 둔다
- D. DynamoDB 테이블에 주문을 넣고 처리 서버가 1초마다 스캔한다

**Q9.** 운영 중인 RDS for MySQL 인스턴스가 **암호화 없이** 만들어졌다. 규정 때문에 저장 데이터 암호화가 필요하다. 올바른 절차는?

- A. 스냅숏을 만들고, 스냅숏을 KMS 키로 **암호화하여 복사**한 뒤, 암호화된 스냅숏에서 새 인스턴스를 복원하고 전환한다
- B. 암호화된 읽기 전용 복제본을 만들고 승격한다
- C. EBS 기본 암호화를 켜면 기존 RDS 스토리지도 암호화된다
- D. 인스턴스 수정(Modify)에서 암호화 옵션을 켠다

**Q10.** 금융 규정상 암호화 키를 **고객 전용(single-tenant) HSM**에 보관하고, HSM의 사용자와 키를 **고객이 직접 관리**해야 하며, 애플리케이션은 PKCS#11 표준 API로 키를 써야 한다. 알맞은 서비스는?

- A. AWS CloudHSM
- B. AWS KMS의 고객 관리형 키(기본 키 저장소)
- C. AWS Secrets Manager
- D. AWS KMS의 AWS 관리형 키

**Q11.** 회사는 한 AWS 계정에서 세 부서가 리소스를 함께 쓴다. 월말에 **부서별 비용**을 보고해야 한다. 필요한 조치를 **2개** 고르시오.

- A. CloudWatch 대시보드를 부서별로 만든다
- B. IAM 사용자 이름에 부서명을 넣는다
- C. 리소스에 `Department` 태그를 일관되게 붙인다
- D. 결제 콘솔에서 `Department` 태그를 비용 할당 태그로 활성화하고 Cost Explorer에서 태그로 그룹화한다
- E. 부서마다 VPC를 따로 만든다

**Q12.** 이미지 변환 워커(Auto Scaling 그룹)가 SQS 큐를 처리한다. CPU 사용률로 확장하니 큐가 쌓여도 늦게 늘어난다. 처리 지연을 일정하게 유지하려면 어떤 지표로 확장해야 하는가?

- A. 인스턴스당 백로그(큐의 보이는 메시지 수 ÷ 실행 중 인스턴스 수)를 목표 값으로 하는 대상 추적 정책
- B. 매시 정각 예약 정책
- C. ALB 요청 수 기준 대상 추적
- D. 네트워크 입력 바이트 기준 단계 정책

**Q13.** 보안팀은 계정에서 **비정상적인 API 호출, 알려진 악성 IP와의 통신, 암호화폐 채굴 의심 트래픽**을 별도 에이전트 설치 없이 탐지하고 싶다. 가장 알맞은 서비스는?

- A. Amazon GuardDuty
- B. AWS WAF
- C. AWS Shield Standard
- D. Amazon Macie

**Q14.** 전자상거래 사이트의 클릭 이벤트(초당 5만 건)를 실시간 추천 엔진과 이상 거래 탐지기가 **동시에** 읽어야 하고, 장애 후 **지난 24시간 데이터를 다시 처리**할 수 있어야 한다. 알맞은 서비스는?

- A. Amazon Kinesis Data Streams
- B. Amazon Data Firehose
- C. Amazon SNS
- D. Amazon SQS 표준 큐

**Q15.** 모바일 앱 사용자가 이메일로 가입·로그인한 뒤, 자기 폴더(`users/<사용자ID>/`)의 S3 객체만 직접 올리고 받을 수 있어야 한다. 서버를 따로 두지 않으려 한다. 알맞은 조합은?

- A. API Gateway API 키를 사용자마다 발급한다
- B. S3 버킷을 퍼블릭으로 열고 앱에서 경로를 숨긴다
- C. Cognito 사용자 풀로 가입·로그인하고, Cognito 자격 증명 풀로 임시 AWS 자격 증명을 받아 사용자 ID 기반 IAM 정책으로 접근을 제한한다
- D. 사용자마다 IAM 사용자를 만들고 액세스 키를 앱에 내려 준다

**Q16.** 스타트업 CFO는 이번 달 AWS 비용이 **월말에 5,000달러를 넘을 것으로 예측되면** 미리 이메일을 받고 싶다. 알맞은 방법은?

- A. Cost Explorer에서 월별 그래프를 매일 확인한다
- B. CloudTrail에서 결제 이벤트를 기록한다
- C. AWS Budgets에서 월 5,000달러 비용 예산을 만들고 예측(forecasted) 비용 100% 알림을 설정한다
- D. Compute Optimizer 추천을 매주 확인한다

**Q17.** 대형 공연 예매 사이트(CloudFront + ALB)는 예매 오픈 때 대규모 DDoS 공격을 받은 적이 있다. **DDoS 대응 전문팀 지원**과 공격으로 늘어난 **확장 비용 보호**를 원한다. 알맞은 서비스는?

- A. AWS Shield Standard
- B. AWS Shield Advanced
- C. AWS Firewall Manager만 사용
- D. Amazon GuardDuty

**Q18.** 특정 IP 대역(203.0.113.0/24)에서 공격성 트래픽이 들어온다. 퍼블릭 서브넷에 있는 **모든 인스턴스로 가는** 이 대역의 트래픽을 즉시 **거부**하려 한다. 가장 알맞은 방법은?

- A. 퍼블릭 서브넷의 네트워크 ACL에 해당 대역을 거부하는 인바운드 규칙을 허용 규칙보다 낮은 번호로 추가한다
- B. 라우팅 테이블에서 해당 대역을 블랙홀로 보낸다
- C. VPC 흐름 로그를 켠다
- D. 모든 인스턴스의 보안 그룹에 해당 대역 거부 규칙을 추가한다

**Q19.** 상품 정보가 바뀌면 ① 검색 인덱스 갱신 ② 캐시 무효화 ③ 추천 모델 재계산 세 작업이 **각자 독립적으로** 실행되어야 한다. 한 작업이 실패해도 다른 작업은 영향을 받으면 안 되고, 실패한 작업은 나중에 다시 처리되어야 한다. 알맞은 구성은?

- A. SNS 토픽에 작업별 SQS 큐 세 개를 구독시키고 각 큐에 DLQ를 둔다
- B. SQS 큐 하나에 넣고 세 워커가 함께 폴링한다
- C. Lambda 하나에서 세 작업을 순서대로 호출한다
- D. 세 작업을 같은 EC2에서 cron으로 돌린다

**Q20.** 증권사는 거래 기록을 **7년 동안 누구도(루트 사용자 포함) 수정·삭제할 수 없게** 보관해야 한다. 알맞은 S3 설정은?

- A. 버킷 정책으로 `s3:DeleteObject`를 거부한다
- B. MFA Delete만 켠다
- C. 버전 관리를 켠 버킷에 S3 Object Lock **규정 준수(compliance) 모드**로 7년 보존 기간을 설정한다
- D. Object Lock **거버넌스(governance) 모드**로 7년 보존 기간을 설정한다

**Q21.** 실시간 입찰 서비스가 DynamoDB에서 같은 항목을 초당 수십만 번 읽으며, 읽기 지연을 **마이크로초 수준**으로 낮춰야 한다. 애플리케이션 코드 변경은 최소화하고 싶다. 알맞은 방법은?

- A. ElastiCache를 두고 캐시 로직을 직접 작성한다
- B. DynamoDB Accelerator(DAX) 클러스터를 두고 DAX 클라이언트로 바꾼다
- C. DynamoDB Streams를 켠다
- D. 읽기 용량 단위를 10배로 늘린다

**Q22.** 매일 밤 4시간 도는 데이터 변환 작업은 재시도가 가능하고 마감도 여유롭다. 비용을 가장 많이 줄이는 방법은?

- A. 온디맨드 용량 예약을 만든다
- B. Dedicated Instances를 쓴다
- C. 온디맨드 인스턴스를 1년 예약한다
- D. 여러 인스턴스 유형을 허용한 Spot 인스턴스로 실행한다

**Q23.** 여러 마이크로서비스가 Aurora PostgreSQL 비밀번호를 환경 변수에 넣어 쓰고 있다. 보안팀은 비밀번호를 **30일마다 자동 교체**하고 애플리케이션이 항상 최신 값을 읽기를 원한다. 운영 부담이 가장 적은 방법은?

- A. 비밀번호를 AMI에 넣고 매달 새 AMI를 배포한다
- B. Systems Manager Parameter Store 표준 파라미터에 저장하고 cron으로 교체 스크립트를 돌린다
- C. 비밀번호를 S3에 KMS로 암호화해 두고 매달 사람이 바꾼다
- D. AWS Secrets Manager에 비밀을 저장하고 관리형 교체(rotation)를 30일 주기로 켠 뒤, 애플리케이션이 Secrets Manager에서 값을 읽게 한다

**Q24.** 운영팀은 수백 대의 EC2 중 **과다 프로비저닝된** 인스턴스를 찾아, 사용률 기록을 바탕으로 알맞은 인스턴스 유형을 추천받고 싶다. 알맞은 서비스는?

- A. AWS Compute Optimizer
- B. AWS Cost and Usage Report
- C. AWS Budgets
- D. AWS Trusted Advisor의 보안 검사

**Q25.** 글로벌 결제 서비스는 Aurora MySQL을 쓴다. 리전 전체 장애 시 **RPO 수 초, RTO 수 분**으로 다른 리전에서 서비스를 이어가야 한다. 가장 알맞은 방법은?

- A. Multi-AZ RDS for MySQL로 바꾼다
- B. 같은 리전에 Aurora 복제본을 3개 추가한다
- C. Aurora Global Database로 보조 리전을 구성한다
- D. 매시간 스냅숏을 다른 리전으로 복사한다

**Q26.** 정적 웹사이트를 S3 + CloudFront로 서비스한다. 사용자가 S3 객체 URL로 **직접** 접근하는 것은 막고 CloudFront를 통해서만 읽게 하려 한다. AWS가 권장하는 방법은?

- A. S3 정적 웹사이트 엔드포인트를 오리진으로 쓰고 Referer 헤더로 확인한다
- B. 버킷에 S3 Object Lock을 켠다
- C. CloudFront 배포에 오리진 접근 제어(OAC)를 설정하고, 버킷 정책에서 해당 배포만 `s3:GetObject`를 허용한다
- D. 버킷을 퍼블릭으로 두고 CloudFront 지역 제한을 켠다

**Q27.** 대형 Oracle 데이터베이스가 EC2에서 운영되며 **볼륨 하나에 10만 IOPS 이상**과 밀리초 미만의 일관된 지연이 필요하다. 알맞은 EBS 볼륨 유형은?

- A. st1
- B. io2 Block Express
- C. sc1
- D. gp2

**Q28.** 서버리스 API(Lambda)가 트래픽 급증 때 RDS for PostgreSQL 연결을 다 써 버려 오류가 난다. 알맞은 해결책은?

- A. API Gateway 캐시를 켠다
- B. Lambda 메모리를 늘린다
- C. RDS Proxy로 연결 풀링을 한다
- D. RDS를 Multi-AZ로 바꾼다

**Q29.** 프라이빗 서브넷의 배치 서버가 같은 리전 S3 버킷에 접근해야 한다. 보안 정책상 **인터넷 경로(NAT 포함)를 쓰면 안 된다**. 가장 알맞은 구성은?

- A. NAT 게이트웨이를 만들고 보안 그룹으로 S3 IP만 허용한다
- B. S3 게이트웨이 VPC 엔드포인트를 만들고 프라이빗 서브넷 라우팅 테이블에 연결한다
- C. VPC 피어링으로 S3 VPC와 연결한다
- D. 서버에 탄력적 IP를 붙이고 버킷 정책에서 그 IP만 허용한다

**Q30.** 병원 예약 시스템이 단일 AZ RDS for PostgreSQL에서 운영된다. AZ 장애 시 **사람 개입 없이** 몇 분 안에 DB가 복구되어야 한다. 가장 알맞은 방법은?

- A. 자동 백업 보존 기간을 35일로 늘린다
- B. 같은 AZ에 읽기 전용 복제본을 하나 더 만든다
- C. Multi-AZ 배포로 변경한다
- D. 매시간 수동 스냅숏을 찍는다

**Q31.** 개발·테스트용 EC2 30대와 RDS 5대는 평일 09:00~19:00에만 쓰인다. 지금은 24시간 켜져 있다. 가장 간단하게 비용을 줄이는 방법은?

- A. EventBridge Scheduler(또는 AWS Instance Scheduler 솔루션)로 업무 시간 외에 EC2·RDS를 자동 중지·시작한다
- B. 모든 인스턴스를 3년 RI로 구매한다
- C. 인스턴스를 Dedicated Hosts로 옮긴다
- D. 인스턴스 크기를 두 배로 늘려 대수를 줄인다

**Q32.** 방송사는 원본 영상 아카이브(500TB)를 법적으로 10년 보관해야 한다. 연 1~2회 조회하며 조회 시 **48시간** 안에만 받으면 된다. 가장 저렴한 스토리지는?

- A. S3 Standard-IA
- B. S3 Glacier Deep Archive
- C. S3 Glacier Instant Retrieval
- D. EBS sc1 볼륨

**Q33.** 유체 해석 시뮬레이션은 노드끼리 메시지를 매우 자주 주고받는 **긴밀히 결합된(tightly coupled)** MPI 작업이다. 노드 간 지연을 최소화하고 처리량을 최대화하려면?

- A. 파티션 배치 그룹을 쓴다
- B. 여러 AZ에 고르게 분산한다
- C. 분산(spread) 배치 그룹을 쓴다
- D. 같은 AZ의 클러스터 배치 그룹에 EFA를 지원하는 인스턴스를 배치한다

**Q34.** AWS Organizations로 40개 계정을 운영하는 회사가 데이터 주권 규정 때문에 **모든 멤버 계정에서** 서울(ap-northeast-2) 외 리전의 리소스 생성을 막아야 한다. 각 계정의 관리자도 이 제한을 풀 수 없어야 한다. 가장 알맞은 방법은?

- A. `aws:RequestedRegion` 조건으로 서울 외 리전을 거부하는 SCP를 루트(또는 해당 OU)에 연결한다(글로벌 서비스는 예외 처리)
- B. AWS Config 규칙으로 서울 외 리전 리소스를 찾아 알림을 보낸다
- C. CloudTrail로 서울 외 리전 API 호출을 기록한다
- D. 각 계정의 모든 IAM 사용자에게 리전 제한 IAM 정책을 붙인다

**Q35.** 회사는 사내 IdP(SAML 2.0 지원)로 직원 계정을 관리한다. 30개 AWS 계정에 대해 직원이 **한 번 로그인**으로 부서별 권한을 받게 하고, 계정마다 IAM 사용자를 만들지 않으려 한다. 알맞은 서비스는?

- A. 각 계정에 IAM 사용자를 만들고 같은 비밀번호를 쓰게 한다
- B. Amazon Cognito 사용자 풀을 만든다
- C. AWS Directory Service Simple AD를 만든다
- D. AWS IAM Identity Center를 사내 IdP와 연동하고 권한 세트를 계정·그룹에 할당한다

**Q36.** 서울 리전에 있는 뉴스 사이트의 이미지·CSS·JS를 전 세계 독자에게 **낮은 지연**으로 제공하고 싶다. 가장 알맞은 방법은?

- A. CloudFront 배포를 만들어 정적 파일을 엣지에서 캐시한다
- B. 모든 리전에 EC2 웹 서버를 하나씩 띄운다
- C. S3 Transfer Acceleration을 켠다
- D. 더 큰 EC2 인스턴스로 바꾼다

**Q37.** ALB 뒤의 쇼핑몰 웹 애플리케이션이 SQL 인젝션과 크로스 사이트 스크립팅(XSS) 공격을 받고 있다. 애플리케이션 코드를 고치기 전까지 **요청 단계에서** 막고 싶다. 가장 알맞은 서비스는?

- A. Amazon GuardDuty를 켠다
- B. AWS WAF 웹 ACL에 SQL 인젝션·XSS 관리형 규칙을 넣고 ALB에 연결한다
- C. 보안 그룹에서 공격자 IP를 거부한다
- D. AWS Shield Advanced를 구독한다

**Q38.** 주식 시세 수신 서비스는 TCP로 **초당 수백만 건**의 요청을 받고, 클라이언트 원본 IP를 그대로 서버에서 봐야 하며, 방화벽 등록을 위해 **AZ별 고정 IP**가 필요하다. 알맞은 로드 밸런서는?

- A. Gateway Load Balancer
- B. Network Load Balancer(AZ별 탄력적 IP 할당)
- C. Classic Load Balancer
- D. Application Load Balancer

**Q39.** 감사 법인이 자기 AWS 계정에서 고객사 계정의 CloudTrail 로그 버킷을 읽어야 한다. 고객사는 감사 법인 직원마다 IAM 사용자를 만들어 주고 싶지 않다. 가장 알맞은 방법은?

- A. 버킷을 퍼블릭 읽기로 바꾸고 감사가 끝나면 되돌린다
- B. 감사 법인 계정에 IAM 그룹을 만들고 고객사 버킷 ARN을 넣는다
- C. 고객사 루트 사용자의 액세스 키를 감사 법인에 전달한다
- D. 고객사 계정에 읽기 권한 IAM 역할을 만들고, 신뢰 정책에서 감사 법인 계정을 주체로 허용해 역할을 맡게(AssumeRole) 한다

**Q40.** 운영팀은 프라이빗 서브넷 EC2에 접속하기 위해 배스천 호스트를 두고 22번 포트를 열어 두었다. **인바운드 포트를 하나도 열지 않고**, 접속 기록을 남기면서 셸에 들어가고 싶다. 알맞은 방법은?

- A. SSM 에이전트와 적절한 IAM 역할을 갖춘 인스턴스에 Systems Manager Session Manager로 접속하고 세션 로그를 S3·CloudWatch Logs에 남긴다
- B. 모든 인스턴스에 퍼블릭 IP를 붙이고 키 페어를 매달 교체한다
- C. Site-to-Site VPN을 연결하고 22번 포트를 VPN 대역에 연다
- D. 배스천 호스트의 보안 그룹을 회사 IP로 좁힌다

**Q41.** 사진 편집 서비스 '포토밍'의 EC2 인스턴스가 S3 버킷에 이미지를 올린다. 현재는 개발자가 만든 IAM 사용자의 액세스 키를 서버 설정 파일에 넣어 두었다. 보안 감사에서 지적을 받았다. 가장 안전한 개선 방법은?

- A. S3 버킷 정책에서 EC2 인스턴스의 퍼블릭 IP만 허용한다
- B. 필요한 S3 권한만 가진 IAM 역할을 만들고 인스턴스 프로필로 EC2에 연결한다
- C. 액세스 키를 90일마다 교체하는 Lambda 함수를 만든다
- D. 액세스 키를 AWS Secrets Manager에 저장하고 애플리케이션이 시작할 때 읽게 한다

**Q42.** 사내 설문 시스템의 DB는 한 달에 며칠만 집중적으로 쓰이고, 나머지 기간에는 거의 유휴 상태다. 사용 시기는 예측하기 어렵다. 관계형 DB가 필요하다. 가장 비용 효율적인 선택은?

- A. Aurora 프로비저닝 인스턴스 3년 RI
- B. RDS for MySQL Multi-AZ 대형 인스턴스
- C. Aurora Serverless v2(최소 용량을 낮게, 필요하면 자동 일시 중지)
- D. EC2에 MySQL을 설치해 24시간 운영

**Q43.** 본사와 AWS를 Direct Connect 1회선으로 연결했다. 회선 장애 시에도 **저비용으로** 연결이 유지되어야 하며, 장애 시 대역폭이 줄어도 괜찮다. 알맞은 방법은?

- A. CloudFront로 본사 트래픽을 보낸다
- B. 같은 VPC(가상 프라이빗 게이트웨이 또는 Transit Gateway)에 Site-to-Site VPN을 백업으로 구성한다
- C. 다른 리전에 VPC를 복제한다
- D. 같은 위치에 Direct Connect 회선을 하나 더 만든다

**Q44.** Auto Scaling 그룹의 웹 서버가 사용자 세션을 **로컬 메모리**에 보관한다. 축소(scale-in)나 인스턴스 교체 때 사용자가 로그아웃된다. 고가용성을 유지하며 고치는 방법은?

- A. Auto Scaling 그룹의 축소를 끈다
- B. 인스턴스를 더 큰 유형으로 바꾼다
- C. 세션을 ElastiCache(Valkey/Redis OSS, Multi-AZ) 또는 DynamoDB 같은 외부 저장소로 옮긴다
- D. ALB 고정 세션(sticky session)만 켠다

**Q45.** 쇼핑몰 RDS for MySQL에서 매일 오후 경영 보고서 쿼리가 돌 때 주문 처리 쓰기 지연이 커진다. 보고서는 몇 분 늦은 데이터여도 된다. 가장 알맞은 방법은?

- A. Multi-AZ를 켜고 보고서를 대기 인스턴스에서 돌린다
- B. 읽기 전용 복제본을 만들고 보고서 도구가 복제본에 연결하게 한다
- C. 보고서 시간에 쓰기를 중지한다
- D. 인스턴스 크기를 두 배로 늘린다

**Q46.** 유통 회사가 수백 TB의 판매 데이터를 대상으로 **복잡한 조인과 집계**를 하는 BI 대시보드를 매일 수백 명이 쓴다. 일관된 빠른 쿼리 성능이 필요하다. 알맞은 서비스는?

- A. Amazon RDS for MySQL
- B. Amazon Redshift
- C. Amazon ElastiCache
- D. Amazon DynamoDB

**Q47.** 여러 팀이 쓰는 데이터 레이크 버킷은 객체마다 접근 빈도가 제각각이고 예측할 수 없다. 객체는 대부분 수 MB 이상이다. 운영 부담 없이 저장 비용을 줄이려면?

- A. 매달 접근 로그를 분석해 수동으로 클래스를 바꾼다
- B. 모든 객체를 S3 One Zone-IA로 옮긴다
- C. 모든 객체를 S3 Standard-IA로 옮긴다
- D. S3 Intelligent-Tiering을 사용한다

**Q48.** 사내 위키 시스템의 DR 요구는 **RPO 24시간, RTO 24시간**이며 비용을 최소화해야 한다. 서비스는 EC2 한 대와 RDS 하나로 이뤄져 있다. 가장 알맞은 DR 전략은?

- A. AWS Backup으로 매일 백업하고 백업을 다른 리전으로 복사해 두었다가, 재해 시 그 리전에서 복원한다(백업·복원)
- B. 다른 리전에 축소된 환경을 항상 켜 두고 재해 시 확장한다(웜 스탠바이)
- C. Aurora Global Database로 이전한다
- D. 다른 리전에 같은 규모의 환경을 항상 켜 둔다(다중 사이트 활성-활성)

**Q49.** 애플리케이션 로그는 첫 30일은 자주 보고, 이후 90일까지는 가끔(월 1회) 보며, 그 뒤로는 1년 동안 보관만 하다 삭제한다. 보관 중 조회는 몇 시간 대기해도 된다. 가장 비용 효율적인 수명 주기 규칙은?

- A. 처음부터 S3 Glacier Flexible Retrieval에 저장
- B. 30일 후 S3 One Zone-IA → 90일 후 삭제
- C. 모든 로그를 S3 Standard에 두고 455일 후 만료
- D. 30일 후 S3 Standard-IA → 90일 후 S3 Glacier Flexible Retrieval → 455일 후 만료

**Q50.** 반도체 설계 회사가 S3에 있는 수십 TB 설계 데이터를 수백 대의 인스턴스로 동시에 시뮬레이션한다. 파일 시스템 인터페이스로 **초당 수백 GB의 처리량**이 필요하다. 알맞은 서비스는?

- A. Amazon FSx for Windows File Server
- B. Amazon S3 Glacier Instant Retrieval
- C. Amazon FSx for Lustre(S3 데이터 리포지토리 연결)
- D. Amazon EFS

**Q51.** 보안팀은 S3에 쌓인 ALB 접근 로그를 가끔 SQL로 조회하고 싶다. 서버나 클러스터를 운영하고 싶지 않다. 알맞은 서비스는?

- A. Amazon Redshift 프로비저닝 클러스터
- B. Amazon Athena
- C. Amazon EMR 클러스터
- D. Amazon RDS for PostgreSQL에 로그를 적재

**Q52.** 사내 근태 시스템은 매일 오전 8시 50분~9시 10분에 접속이 몰린다. 지금은 CPU 대상 추적 정책만 있어 9시 직후 몇 분 동안 느리다. 가장 간단한 개선책은?

- A. 8시 40분에 최소 용량을 올리고 9시 30분에 되돌리는 예약된(scheduled) 조정을 추가한다
- B. 인스턴스를 Spot으로 바꾼다
- C. 대상 추적의 목표 CPU를 90%로 올린다
- D. ALB를 NLB로 바꾼다

**Q53.** 공공 데이터 기관이 수십 TB의 위성 이미지를 S3로 다른 기관(모두 AWS 계정 보유)과 공유한다. 저장 비용은 기관이 내되, **다운로드에 따른 요청·전송 비용은 받아 가는 쪽이** 부담하게 하려 한다. 알맞은 기능은?

- A. S3 Object Lambda
- B. CloudFront 서명된 쿠키
- C. S3 Transfer Acceleration
- D. S3 Requester Pays

**Q54.** 의료 데이터 버킷의 요구 사항: ① 저장 시 암호화, ② **누가 언제 키를 사용했는지 감사 기록**, ③ 키 사용 권한을 버킷 권한과 **분리**해 관리. 이 요구를 만족하는 조합을 **2개** 고르시오.

- A. 버킷 정책에 `aws:SecureTransport` 조건만 추가한다
- B. 버킷 기본 암호화를 SSE-S3로 한다
- C. 클라이언트 측에서 하드코딩한 AES 키로 암호화한다
- D. 버킷 기본 암호화를 SSE-KMS로 하고 **고객 관리형 KMS 키**를 지정한다
- E. KMS 키 정책에서 복호화를 허용할 역할을 따로 지정하고, CloudTrail로 KMS API 호출을 기록한다

**Q55.** 콘텐츠 관리 시스템의 Linux 웹 서버들이 3개 AZ에 퍼져 있고, 업로드 파일을 공유한다. 현재는 AZ-a의 NFS 서버(EC2) 한 대가 파일을 제공해 **단일 장애 지점**이다. 운영 부담을 줄이며 고치려면?

- A. Amazon EFS(리전 스토리지 클래스)로 옮기고 각 AZ에 마운트 대상을 만든다
- B. NFS 서버에 io2 EBS 볼륨을 붙인다
- C. EFS One Zone으로 옮긴다
- D. 각 웹 서버에 EBS를 붙이고 rsync로 동기화한다

**Q56.** 남미·아프리카의 사용자들이 서울 리전 S3 버킷으로 수 GB의 동영상을 올리는데 매우 느리다. 버킷 위치는 바꿀 수 없다. 업로드를 빠르게 하는 방법은? (2개 선택)

- A. S3 버전 관리를 켠다
- B. S3 Transfer Acceleration을 켜고 가속 엔드포인트로 업로드한다
- C. S3 Intelligent-Tiering으로 저장한다
- D. 버킷에 Requester Pays를 켠다
- E. 멀티파트 업로드로 파일을 나눠 병렬 전송한다

**Q57.** 모바일 게임은 미국·유럽·아시아 사용자가 **각자 가까운 리전에서 읽고 쓰며**, 한 리전이 장애 나도 다른 리전에서 계속 쓰기가 되어야 한다. 데이터 모델은 키-값이다. 알맞은 서비스는?

- A. Aurora Global Database
- B. 단일 리전 DynamoDB + DAX
- C. ElastiCache 글로벌 데이터 스토어만 사용
- D. DynamoDB 글로벌 테이블

**Q58.** 음성 통화 앱의 미디어 서버는 **UDP**를 쓰고 두 리전의 NLB 뒤에 있다. 전 세계 사용자의 지연과 패킷 손실을 줄이고, 클라이언트에 **고정 IP**를 내려 주고 싶다. 알맞은 서비스는?

- A. Application Load Balancer
- B. Amazon CloudFront
- C. Route 53 지리 근접 라우팅
- D. AWS Global Accelerator

**Q59.** 회사 웹사이트는 서울 리전 ALB에서 운영되고, 장애 시 버지니아 리전 S3의 정적 "점검 중" 페이지를 보여 주려 한다. 가장 알맞은 방법은?

- A. ALB에 S3를 대상으로 추가한다
- B. Route 53 가중치 라우팅으로 50:50을 준다
- C. Route 53 장애 조치(failover) 라우팅으로 기본 레코드는 ALB(상태 확인 연동), 보조 레코드는 S3 정적 웹사이트로 설정한다
- D. CloudFront 지역 제한을 켠다

**Q60.** 회사가 수백 개 S3 버킷 중 어디에 주민등록번호·카드 번호 같은 **개인정보**가 있는지 모른다. 자동으로 찾아 보고서를 받고 싶다. 알맞은 서비스는?

- A. Amazon Detective
- B. AWS Config
- C. Amazon Inspector
- D. Amazon Macie

**Q61.** 규정상 S3에 올라오는 **새 계약서**를 자동으로 다른 리전 버킷에도 사본으로 두어야 한다. 알맞은 방법은? (2개 선택)

- A. CloudFront 배포를 대상 리전에 만든다
- B. 수명 주기 규칙으로 다른 리전으로 전환한다
- C. S3 Transfer Acceleration을 켠다
- D. 원본 버킷에 리전 간 복제(CRR) 규칙과 복제용 IAM 역할을 설정한다
- E. 원본·대상 버킷 모두 버전 관리를 켠다

**Q62.** 라이선스 제약으로 **항상 정확히 한 대**만 실행해야 하는 레거시 서버가 있다. 인스턴스나 AZ에 장애가 나도 **자동으로 다른 AZ에** 새 인스턴스가 떠야 한다. 가장 알맞은 방법은?

- A. CloudWatch 경보로 운영자에게 이메일만 보낸다
- B. Dedicated Host에 인스턴스를 고정한다
- C. 두 AZ 서브넷에 걸친 Auto Scaling 그룹을 최소·최대·원하는 용량 1로 만든다
- D. 인스턴스 두 대를 띄우고 하나는 중지해 둔다

**Q63.** 비용 보고서에서 **NAT 게이트웨이 데이터 처리 요금**이 급증했다. 원인을 보니 프라이빗 서브넷의 애플리케이션이 같은 리전 DynamoDB 테이블에 대량으로 쓰고 있었다. 가장 비용 효율적인 해결책은?

- A. DynamoDB 인터페이스 엔드포인트를 AZ마다 만든다
- B. 애플리케이션을 퍼블릭 서브넷으로 옮긴다
- C. NAT 게이트웨이를 AZ마다 하나씩 더 만든다
- D. DynamoDB 게이트웨이 VPC 엔드포인트를 만들고 프라이빗 서브넷 라우팅 테이블에 연결한다

**Q64.** SQS 큐를 소비하는 워커가 특정 형식의 메시지에서 계속 실패해, 그 메시지가 수천 번 다시 처리되며 다른 메시지 처리를 방해한다. 가장 알맞은 조치는?

- A. 가시성 제한 시간을 0초로 줄인다
- B. 배달 못한 편지 큐(DLQ)를 만들고 원본 큐의 재드라이브 정책에서 `maxReceiveCount`를 정한다
- C. 메시지 보존 기간을 1분으로 줄인다
- D. 큐를 FIFO 큐로 바꾼다

**Q65.** 보험 청구 처리는 ① 서류 검증 ② 사기 점수 계산 ③ 담당자 승인(최대 3일 대기) ④ 지급 순서로 진행되고, 단계마다 재시도·실패 처리가 필요하다. 코드로 상태를 관리하고 싶지 않다. 알맞은 서비스는?

- A. SQS 큐 네 개를 연결하고 각 단계에서 상태를 DynamoDB에 기록
- B. EventBridge 규칙 하나
- C. Lambda 함수 하나에서 모든 단계를 순서대로 호출
- D. AWS Step Functions Standard 워크플로(사람 승인은 작업 토큰 콜백)


> ⏱️ **여기까지가 시험입니다.** 남은 시간에 `?` 표시한 문항을 검토하고, 시간이 끝나면 펜을 내려놓으세요.

---

## 4. 채점 전에

- 빈칸으로 둔 문항이 있다면 지금이라도 **찍지 말고 빈칸 그대로** 채점하세요. 실제 시험에서는 빈칸이 오답이므로, 다음 번에는 시간 안에 반드시 찍는 연습을 합니다.
- `?` 표시 문항 중 맞힌 것도 해설을 꼭 읽으세요. **운으로 맞힌 문항**이 실제 시험에서 가장 위험합니다.

---

## 5. 정답표

| 문항 | 정답 | 도메인 | 주제 | 문항 | 정답 | 도메인 | 주제 |
|---|---|---|---|---|---|---|---|
| Q1 | **B** | 비용 최적화 | 변화가 예정된 꾸준한 컴퓨팅 | Q34 | **A** | 보안 | 조직 전체 리전 제한 |
| Q2 | **D** | 복원력 | 웹 계층 다중 AZ | Q35 | **D** | 보안 | 다중 계정 SSO |
| Q3 | **C** | 비용 최적화 | 꾸준한 DynamoDB 트래픽 | Q36 | **A** | 고성능 | 전 세계 정적 콘텐츠 |
| Q4 | **C** | 보안 | 다른 VPC의 SaaS에 비공개 연결 | Q37 | **B** | 보안 | SQL 인젝션 방어 |
| Q5 | **C** | 복원력 | 백업 중앙 관리 | Q38 | **B** | 고성능 | L4 초고성능 |
| Q6 | **A** | 보안 | ALB 인증서 | Q39 | **D** | 보안 | 교차 계정 접근 |
| Q7 | **B** | 복원력 | 순서·중복 없는 처리 | Q40 | **A** | 보안 | SSH 포트 없이 접속 |
| Q8 | **B** | 복원력 | 급증 흡수 | Q41 | **B** | 보안 | EC2 → S3 권한 |
| Q9 | **A** | 보안 | 기존 RDS 암호화 | Q42 | **C** | 비용 최적화 | 간헐적 DB |
| Q10 | **A** | 보안 | 전용 HSM | Q43 | **B** | 복원력 | 하이브리드 연결 이중화 |
| Q11 | **C,D** | 비용 최적화 | 부서별 비용 배분 (2개 선택) | Q44 | **C** | 복원력 | 세션 상태 외부화 |
| Q12 | **A** | 고성능 | 큐 기반 확장 | Q45 | **B** | 고성능 | 보고서 쿼리 분리 |
| Q13 | **A** | 보안 | 위협 탐지 | Q46 | **B** | 고성능 | 데이터 웨어하우스 |
| Q14 | **A** | 고성능 | 클릭스트림 실시간 처리 | Q47 | **D** | 비용 최적화 | 접근 패턴을 모를 때 |
| Q15 | **C** | 보안 | 모바일 앱 사용자 인증 | Q48 | **A** | 복원력 | 저비용 DR |
| Q16 | **C** | 비용 최적화 | 예산 초과 예측 알림 | Q49 | **D** | 비용 최적화 | 로그 수명 주기 |
| Q17 | **B** | 보안 | DDoS 대응 | Q50 | **C** | 고성능 | HPC 공유 파일 시스템 |
| Q18 | **A** | 보안 | 서브넷 단위 IP 차단 | Q51 | **B** | 고성능 | S3 로그 즉석 분석 |
| Q19 | **A** | 복원력 | 팬아웃 | Q52 | **A** | 고성능 | 예측 가능한 급증 |
| Q20 | **C** | 보안 | WORM 보존 | Q53 | **D** | 비용 최적화 | 다운로드 비용 전가 |
| Q21 | **B** | 고성능 | DynamoDB 마이크로초 읽기 | Q54 | **D,E** | 보안 | S3 암호화와 키 사용 감사 (2개 선택) |
| Q22 | **D** | 비용 최적화 | 중단 가능한 배치 | Q55 | **A** | 복원력 | 여러 AZ 공유 파일 |
| Q23 | **D** | 보안 | DB 자격 증명 자동 교체 | Q56 | **B,E** | 고성능 | 먼 곳에서 S3 업로드 |
| Q24 | **A** | 비용 최적화 | rightsizing | Q57 | **D** | 복원력 | 다중 리전 활성-활성 NoSQL |
| Q25 | **C** | 복원력 | 리전 간 DB DR | Q58 | **D** | 고성능 | UDP 글로벌 성능 |
| Q26 | **C** | 보안 | S3를 CloudFront로만 | Q59 | **C** | 복원력 | DNS 장애 조치 |
| Q27 | **B** | 고성능 | 고성능 블록 스토리지 | Q60 | **D** | 보안 | S3 민감 데이터 발견 |
| Q28 | **C** | 고성능 | Lambda와 RDS 연결 | Q61 | **D,E** | 복원력 | 객체 리전 간 복사 |
| Q29 | **B** | 보안 | 프라이빗 서브넷 → S3 | Q62 | **C** | 복원력 | 단일 인스턴스 자동 복구 |
| Q30 | **C** | 복원력 | RDS 고가용성 | Q63 | **D** | 비용 최적화 | NAT 처리 요금 |
| Q31 | **A** | 비용 최적화 | 업무 시간에만 쓰는 환경 | Q64 | **B** | 복원력 | 실패 메시지 격리 |
| Q32 | **B** | 비용 최적화 | 장기 아카이브 | Q65 | **D** | 복원력 | 여러 단계 워크플로 |
| Q33 | **D** | 고성능 | HPC 노드 간 지연 |  |  |  |  |

---

## 6. 해설

> 문항 10개씩 접어 두었습니다. 해설은 정답 근거와 **오답이 어긋나는 조건**을 함께 적었고, 📎 링크는 근거가 되는 AWS 공식 문서입니다.

<details>
<summary><b>Q1 ~ Q10 해설</b></summary>

**Q1 — 정답 B** · 비용 최적화 · 변화가 예정된 꾸준한 컴퓨팅

Compute SP는 EC2 패밀리·리전 변경과 **Fargate·Lambda**까지 할인이 적용된다. 패밀리가 바뀔 예정이면 RI·EC2 Instance SP는 할인이 사라질 수 있다.

📎 [Savings Plans 종류](https://docs.aws.amazon.com/savingsplans/latest/userguide/plan-types.html)

**Q2 — 정답 D** · 복원력 · 웹 계층 다중 AZ

여러 AZ에 걸친 Auto Scaling 그룹 + ALB가 **AZ 장애 내성과 자동 교체**의 기본형이다. ELB 상태 확인을 켜면 애플리케이션이 응답하지 않는 인스턴스도 교체한다.

📎 [Auto Scaling과 ELB](https://docs.aws.amazon.com/autoscaling/ec2/userguide/autoscaling-load-balancer.html)

**Q3 — 정답 C** · 비용 최적화 · 꾸준한 DynamoDB 트래픽

트래픽이 예측 가능하고 꾸준하면 **프로비저닝 용량**이 온디맨드보다 싸다. DAX는 읽기 캐시이므로 쓰기 비용을 줄이지 못한다. Standard-IA 테이블 클래스는 **저장 비용**이 큰 테이블용이다.

📎 [DynamoDB 용량 모드](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/capacity-mode.html)

**Q4 — 정답 C** · 보안 · 다른 VPC의 SaaS에 비공개 연결

PrivateLink는 **특정 서비스만** 소비자 VPC의 프라이빗 IP로 노출하며 **CIDR이 겹쳐도** 동작한다. 피어링과 Transit Gateway는 겹치는 CIDR 사이의 라우팅을 할 수 없고, 서비스 하나가 아니라 네트워크 전체를 연결한다.

📎 [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)

**Q5 — 정답 C** · 복원력 · 백업 중앙 관리

AWS Backup은 여러 서비스의 백업을 **정책으로 중앙 관리**하고, Organizations 백업 정책으로 여러 계정에 배포하며, Audit Manager로 준수 보고를 만든다. DLM은 EBS 스냅숏·AMI 중심이다.

📎 [AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)

**Q6 — 정답 A** · 보안 · ALB 인증서

ACM 퍼블릭 인증서는 ALB·CloudFront·API Gateway 같은 통합 서비스에서 **무료**이고, DNS 검증을 쓰면 **자동 갱신**된다. 자체 서명 인증서는 브라우저가 신뢰하지 않는다.

📎 [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)

**Q7 — 정답 B** · 복원력 · 순서·중복 없는 처리

FIFO 큐는 **메시지 그룹 단위 순서**와 중복 제거를 제공한다. 그룹 ID를 계좌별로 두면 계좌 안 순서는 지키면서 계좌끼리는 병렬 처리된다. 그룹 ID를 하나로 고정하면 전체가 직렬화되어 처리량이 무너진다.

📎 [SQS FIFO 큐](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html)

**Q8 — 정답 B** · 복원력 · 급증 흡수

SQS는 생산자와 소비자를 **분리**해 급증을 **버퍼링**하고 메시지를 내구성 있게 보관한다. 소비자는 큐의 적체량(메시지 수)에 따라 확장한다. 로드 밸런서는 버퍼가 아니며 처리 능력을 넘는 요청은 실패한다.

📎 [SQS 기반 확장](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-using-sqs-queue.html)

**Q9 — 정답 A** · 보안 · 기존 RDS 암호화

RDS는 **실행 중인 암호화되지 않은 인스턴스를 직접 암호화할 수 없다**. 스냅숏 복사 시 암호화를 지정하고 그 스냅숏에서 복원하는 것이 표준 절차다. 암호화되지 않은 인스턴스에서 암호화된 읽기 전용 복제본은 만들 수 없다.

📎 [RDS 리소스 암호화](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html)

**Q10 — 정답 A** · 보안 · 전용 HSM

CloudHSM은 고객 VPC에 **전용 HSM 클러스터**를 제공하고 HSM 사용자·키를 고객이 관리하며 PKCS#11·JCE·OpenSSL 같은 표준 인터페이스를 지원한다. KMS도 검증된 HSM을 쓰지만 다중 테넌트 관리형 서비스이며 PKCS#11 인터페이스를 제공하지 않는다.

📎 [AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/introduction.html)

</details>

<details>
<summary><b>Q11 ~ Q20 해설</b></summary>

**Q11 — 정답 C,D** · 비용 최적화 · 부서별 비용 배분 (2개 선택)

태그를 붙이고 **비용 할당 태그로 활성화**하면 Cost Explorer·CUR에서 태그별로 비용을 나눌 수 있다. 장기적으로는 부서별 계정 분리(Organizations)도 좋은 방법이다.

📎 [비용 할당 태그](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)

**Q12 — 정답 A** · 고성능 · 큐 기반 확장

큐 워커는 **인스턴스당 처리해야 할 메시지 수(백로그)** 가 실제 부하를 나타낸다. CloudWatch 지표 수식으로 계산해 대상 추적 정책에 쓰는 것이 권장 패턴이다.

📎 [SQS 기반 확장](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-using-sqs-queue.html)

**Q13 — 정답 A** · 보안 · 위협 탐지

GuardDuty는 CloudTrail 이벤트, VPC 흐름 로그, DNS 로그 등을 분석해 **위협을 탐지**하며 기본 데이터 소스는 에이전트가 필요 없다. WAF·Shield는 차단·완화 쪽이고 Macie는 데이터 분류다.

📎 [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)

**Q14 — 정답 A** · 고성능 · 클릭스트림 실시간 처리

Kinesis Data Streams는 여러 소비자가 같은 스트림을 **독립적으로** 읽고 보존 기간 안에서 **재처리**할 수 있다. SQS는 소비하면 메시지가 사라지고, Firehose는 저장 대상으로 배달하는 서비스다.

📎 [Kinesis Data Streams](https://docs.aws.amazon.com/streams/latest/dev/introduction.html)

**Q15 — 정답 C** · 보안 · 모바일 앱 사용자 인증

사용자 풀은 **가입·로그인·토큰**, 자격 증명 풀은 토큰을 **임시 AWS 자격 증명**으로 바꿔 준다. IAM 정책의 정책 변수(자격 증명 풀의 identity ID)로 사용자별 접두사만 허용할 수 있다. 사용자마다 IAM 사용자를 만드는 것은 확장성과 보안 모두에 나쁘다.

📎 [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)

**Q16 — 정답 C** · 비용 최적화 · 예산 초과 예측 알림

Budgets는 **예측 비용** 기준 알림을 지원한다. Cost Explorer는 보여 주기만 하고 알림을 보내지 않는다.

📎 [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)

**Q17 — 정답 B** · 보안 · DDoS 대응

Shield Advanced는 L3/L4·L7 DDoS 탐지·완화 강화, **Shield Response Team(SRT)** 지원, DDoS로 인한 확장 요금에 대한 **비용 보호**를 제공한다. Shield Standard는 모든 고객에게 무료로 기본 보호만 준다.

📎 [AWS Shield Advanced](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-advanced-summary.html)

**Q18 — 정답 A** · 보안 · 서브넷 단위 IP 차단

네트워크 ACL은 서브넷 단위이고 **거부 규칙**을 지원하며 번호가 낮은 규칙부터 평가한다. 보안 그룹은 허용 규칙만 있다. 라우팅 테이블은 목적지 기반이라 들어오는 출발지를 막지 못하고, 흐름 로그는 기록만 한다.

📎 [네트워크 ACL](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

**Q19 — 정답 A** · 복원력 · 팬아웃

SNS → SQS 팬아웃은 메시지를 작업마다 **복제**해 각자의 큐에서 독립적으로 처리하게 한다. 큐마다 DLQ를 두면 실패한 메시지를 따로 재처리할 수 있다.

📎 [SNS 팬아웃](https://docs.aws.amazon.com/sns/latest/dg/sns-sqs-as-subscriber.html)

**Q20 — 정답 C** · 보안 · WORM 보존

규정 준수 모드에서는 보존 기간 동안 **루트 사용자를 포함해 누구도** 객체 버전을 덮어쓰거나 지울 수 없고 보존 기간을 줄일 수도 없다. 거버넌스 모드는 특별 권한(`s3:BypassGovernanceRetention`)이 있으면 우회할 수 있다. 버킷 정책은 관리자가 바꿀 수 있다.

📎 [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)

</details>

<details>
<summary><b>Q21 ~ Q30 해설</b></summary>

**Q21 — 정답 B** · 고성능 · DynamoDB 마이크로초 읽기

DAX는 DynamoDB API와 호환되는 **인메모리 캐시**로 마이크로초 읽기를 제공하고, SDK 클라이언트만 바꾸면 되므로 코드 변경이 적다. ElastiCache도 빠르지만 캐시 로직(읽기·무효화)을 직접 짜야 한다.

📎 [DynamoDB Accelerator](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)

**Q22 — 정답 D** · 비용 최적화 · 중단 가능한 배치

중단돼도 다시 시도할 수 있는 배치는 **Spot**이 가장 싸다. 하루 4시간만 쓰는 작업에 RI·SP 같은 24시간 약정은 낭비가 크다.

📎 [Spot 인스턴스](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html)

**Q23 — 정답 D** · 보안 · DB 자격 증명 자동 교체

Secrets Manager는 RDS·Aurora 등에 대해 **교체 기능을 내장**하고 있다. Parameter Store는 저장은 되지만 교체를 직접 구현해야 하므로 운영 부담이 크다.

📎 [Secrets Manager 교체](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)

**Q24 — 정답 A** · 비용 최적화 · rightsizing

Compute Optimizer는 CloudWatch 사용률 지표를 분석해 EC2·Auto Scaling 그룹·EBS·Lambda 등의 **rightsizing 추천**을 준다. Budgets는 예산 알림, CUR은 원시 비용 데이터다.

📎 [AWS Compute Optimizer](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html)

**Q25 — 정답 C** · 복원력 · 리전 간 DB DR

Aurora Global Database는 스토리지 수준에서 리전 간 복제(보통 1초 미만 지연)하고, 장애 시 보조 리전을 승격해 **RPO 초·RTO 분 단위**를 만족한다. 같은 리전 복제본·Multi-AZ는 리전 장애를 막지 못하고, 매시간 스냅숏은 RPO가 1시간이다.

📎 [Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html)

**Q26 — 정답 C** · 보안 · S3를 CloudFront로만

OAC는 CloudFront가 서명한 요청으로 S3에 접근하게 하고, 버킷 정책에서 그 배포(`AWS:SourceArn`)만 허용하면 직접 접근이 막힌다. 레거시 OAI보다 OAC가 권장된다. Referer 헤더는 위조할 수 있고, Object Lock은 삭제 방지 기능이다.

📎 [S3 오리진 접근 제한(OAC)](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)

**Q27 — 정답 B** · 고성능 · 고성능 블록 스토리지

io2 Block Express는 가장 높은 IOPS·처리량과 밀리초 미만 지연, 높은 내구성을 제공하는 프로비저닝된 IOPS SSD다. st1·sc1은 HDD로 순차 처리량용이다.

📎 [EBS 볼륨 유형](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)

**Q28 — 정답 C** · 고성능 · Lambda와 RDS 연결

RDS Proxy는 많은 Lambda 실행 환경의 연결을 **풀링**해 DB 연결 수를 안정시킨다.

📎 [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)

**Q29 — 정답 B** · 보안 · 프라이빗 서브넷 → S3

게이트웨이 엔드포인트는 S3·DynamoDB 트래픽을 **AWS 네트워크 안에서** 보내고 인터넷·NAT를 거치지 않는다. 엔드포인트 정책과 버킷 정책(`aws:SourceVpce`)으로 접근을 더 좁힐 수도 있다. S3는 사용자의 VPC에 있는 서비스가 아니므로 피어링 대상이 아니다.

📎 [게이트웨이 엔드포인트](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html)

**Q30 — 정답 C** · 복원력 · RDS 고가용성

Multi-AZ는 다른 AZ에 **동기식 대기 인스턴스**를 두고 장애 시 **자동 장애 조치**(DNS 엔드포인트 전환)한다. 복제본은 승격이 수동이고, 백업·스냅숏은 복원에 시간이 걸린다.

📎 [RDS Multi-AZ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

</details>

<details>
<summary><b>Q31 ~ Q40 해설</b></summary>

**Q31 — 정답 A** · 비용 최적화 · 업무 시간에만 쓰는 환경

주 50시간만 쓰면 나머지 118시간(약 70%)의 컴퓨팅 비용을 중지로 없앨 수 있다. 24시간 약정(RI)은 쓰지 않는 시간에도 돈을 낸다. 중지 중에도 EBS·RDS 스토리지 요금은 나온다.

📎 [Instance Scheduler on AWS](https://docs.aws.amazon.com/solutions/latest/instance-scheduler-on-aws/solution-overview.html)

**Q32 — 정답 B** · 비용 최적화 · 장기 아카이브

Deep Archive는 S3에서 가장 저렴한 클래스이며 표준 조회 12시간 이내, 대량 조회 48시간 이내다. 즉시 조회가 필요 없으므로 Instant Retrieval은 과하다.

📎 [S3 스토리지 클래스](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)

**Q33 — 정답 D** · 고성능 · HPC 노드 간 지연

클러스터 배치 그룹은 인스턴스를 한 AZ 안에서 **물리적으로 가깝게** 배치해 저지연·고대역폭을 준다. EFA(Elastic Fabric Adapter)는 MPI 같은 HPC 통신을 가속한다. 분산 배치는 장애 격리용이다.

📎 [배치 그룹](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)

**Q34 — 정답 A** · 보안 · 조직 전체 리전 제한

SCP는 멤버 계정에서 쓸 수 있는 **최대 권한**을 정하며 계정 관리자(루트 포함)도 넘을 수 없다. IAM 정책은 계정 관리자가 바꿀 수 있고, Config·CloudTrail은 탐지·기록이지 예방이 아니다. IAM·CloudFront 같은 글로벌 서비스는 us-east-1 엔드포인트를 쓰므로 예외를 두는 것이 일반적이다.

📎 [SCP 예: 리전 거부](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples_general.html)

**Q35 — 정답 D** · 보안 · 다중 계정 SSO

IAM Identity Center는 Organizations의 여러 계정에 대한 **직원용 SSO**를 제공하고, 외부 IdP와 연동해 **권한 세트**로 계정별 역할을 만든다. Cognito는 고객(앱 사용자)용 인증이다.

📎 [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)

**Q36 — 정답 A** · 고성능 · 전 세계 정적 콘텐츠

CloudFront는 전 세계 엣지 로케이션에서 콘텐츠를 **캐시**해 사용자 가까이에서 응답한다. Transfer Acceleration은 S3 **업로드** 가속용이다.

📎 [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)

**Q37 — 정답 B** · 보안 · SQL 인젝션 방어

WAF는 HTTP 요청 내용을 검사하는 **L7 방화벽**으로 SQLi·XSS 같은 패턴을 막는다. Shield는 DDoS 방어, GuardDuty는 위협 **탐지**다. 보안 그룹은 거부 규칙이 없고 요청 내용을 보지 않는다.

📎 [AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html)

**Q38 — 정답 B** · 고성능 · L4 초고성능

NLB는 L4에서 초저지연·대량 처리를 하고, **AZ마다 탄력적 IP**를 지정할 수 있으며 대상 유형에 따라 원본 IP를 보존한다. ALB는 고정 IP를 주지 않는다. GWLB는 방화벽 어플라이언스 삽입용이다.

📎 [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)

**Q39 — 정답 D** · 보안 · 교차 계정 접근

교차 계정 접근의 기본은 **신뢰 정책으로 다른 계정을 허용한 역할**이다. 상대 계정의 주체는 STS AssumeRole로 임시 자격 증명을 받는다. 상대 계정 쪽 IAM 정책만으로는 내 계정 리소스 권한이 생기지 않는다(리소스 기반 정책이나 역할이 필요).

📎 [교차 계정 역할 튜토리얼](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)

**Q40 — 정답 A** · 보안 · SSH 포트 없이 접속

Session Manager는 에이전트가 **밖으로** 연결하는 방식이라 인바운드 포트·배스천·SSH 키가 필요 없고, IAM으로 접근을 제어하며 세션 기록을 남길 수 있다. 프라이빗 서브넷이라면 NAT 또는 SSM용 인터페이스 엔드포인트가 필요하다.

📎 [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)

</details>

<details>
<summary><b>Q41 ~ Q50 해설</b></summary>

**Q41 — 정답 B** · 보안 · EC2 → S3 권한

EC2에서 AWS API를 부를 때의 정답은 **IAM 역할(인스턴스 프로필)** 이다. 인스턴스 메타데이터로 자동 교체되는 임시 자격 증명을 받으므로 장기 액세스 키가 사라진다. 키를 Secrets Manager에 넣거나 교체해도 장기 자격 증명이라는 근본 문제는 남는다.

📎 [EC2용 IAM 역할](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

**Q42 — 정답 C** · 비용 최적화 · 간헐적 DB

Aurora Serverless v2는 부하에 따라 ACU를 자동으로 늘리고 줄이며, 유휴 시 최소 용량(0 ACU 자동 일시 중지 포함)으로 내려가 **쓰는 만큼** 과금된다. 유휴가 긴 워크로드에 24시간 약정은 낭비다.

📎 [Aurora Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)

**Q43 — 정답 B** · 복원력 · 하이브리드 연결 이중화

대역폭 감소를 허용하면 **VPN 백업**이 가장 싸다. DX는 BGP로 우선되고 장애 시 VPN으로 넘어간다. 핵심 워크로드라 대역폭을 유지해야 한다면 **다른 DX 위치**에 두 번째 회선을 두는 것이 권장된다.

📎 [Direct Connect 복원력 권장 사항](https://docs.aws.amazon.com/directconnect/latest/UserGuide/resiliency_toolkit.html)

**Q44 — 정답 C** · 복원력 · 세션 상태 외부화

상태를 서버 밖으로 빼야(stateless) 인스턴스가 사라져도 세션이 유지된다. 고정 세션은 같은 서버로 보내 줄 뿐 그 서버가 사라지면 세션도 사라진다.

📎 [ElastiCache 세션 관리](https://aws.amazon.com/caching/session-management/)

**Q45 — 정답 B** · 고성능 · 보고서 쿼리 분리

읽기 전용 복제본은 비동기 복제로 **읽기 부하를 분리**한다(약간의 지연 허용). Multi-AZ 대기 인스턴스는 읽기를 받지 않는다.

📎 [RDS 읽기 전용 복제본](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)

**Q46 — 정답 B** · 고성능 · 데이터 웨어하우스

Redshift는 열 기반 저장과 MPP로 대규모 분석 쿼리에 최적화된 **데이터 웨어하우스**다. DynamoDB는 키-값 OLTP, RDS는 트랜잭션 DB다.

📎 [Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/welcome.html)

**Q47 — 정답 D** · 비용 최적화 · 접근 패턴을 모를 때

Intelligent-Tiering은 객체별 접근을 보고 자동으로 계층을 바꾸며 조회 요금이 없다. 패턴을 모를 때의 기본 답이다.

📎 [S3 Intelligent-Tiering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering-overview.html)

**Q48 — 정답 A** · 복원력 · 저비용 DR

RPO·RTO가 하루 단위면 **백업·복원**이 가장 싸다. 파일럿 라이트 → 웜 스탠바이 → 다중 사이트로 갈수록 RTO·RPO는 줄지만 비용이 커진다.

📎 [AWS의 DR 옵션(백서)](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)

**Q49 — 정답 D** · 비용 최적화 · 로그 수명 주기

자주 → 가끔 → 보관만으로 바뀌는 **알려진 패턴**은 수명 주기 전환이 정답이다. 처음부터 Glacier에 넣으면 자주 보는 30일 동안 조회가 느리고 비싸다. 마지막 보기는 1년 보관 요구를 어긴다.

📎 [S3 수명 주기](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)

**Q50 — 정답 C** · 고성능 · HPC 공유 파일 시스템

FSx for Lustre는 HPC용 병렬 파일 시스템으로 S3와 연결해 데이터를 파일처럼 다루며 매우 높은 처리량을 낸다.

📎 [FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/what-is.html)

</details>

<details>
<summary><b>Q51 ~ Q60 해설</b></summary>

**Q51 — 정답 B** · 고성능 · S3 로그 즉석 분석

Athena는 S3 데이터를 **서버리스로** 표준 SQL 조회하며 스캔한 데이터만큼 과금된다. 가끔 하는 즉석 분석에 맞는다.

📎 [Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)

**Q52 — 정답 A** · 고성능 · 예측 가능한 급증

시각이 정해진 급증은 **예약된 조정**으로 미리 늘리는 것이 가장 간단하다(패턴이 일정하면 예측 조정도 선택지). 목표 CPU를 높이면 오히려 더 늦게 확장된다.

📎 [예약된 조정](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-scheduled-scaling.html)

**Q53 — 정답 D** · 비용 최적화 · 다운로드 비용 전가

Requester Pays 버킷은 요청·다운로드 비용을 **요청자 계정**에 청구한다. 요청자는 인증된 AWS 사용자여야 한다.

📎 [Requester Pays](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RequesterPaysBuckets.html)

**Q54 — 정답 D,E** · 보안 · S3 암호화와 키 사용 감사 (2개 선택)

SSE-KMS + 고객 관리형 키를 쓰면 **키 정책**으로 키 사용 권한을 버킷 권한과 별도로 통제할 수 있고, KMS 호출(Decrypt 등)이 **CloudTrail에 기록**된다. SSE-S3는 키 관리·감사를 AWS가 대신하므로 ②·③을 만족하지 못한다. `aws:SecureTransport`는 전송 중 암호화 강제다.

📎 [SSE-KMS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)

**Q55 — 정답 A** · 복원력 · 여러 AZ 공유 파일

EFS 리전(Standard) 스토리지는 데이터를 **여러 AZ에 저장**하고 AZ마다 마운트 대상을 두어 단일 장애 지점을 없앤다. One Zone은 AZ 하나에만 저장되므로 AZ 장애에 약하다.

📎 [Amazon EFS](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)

**Q56 — 정답 B,E** · 고성능 · 먼 곳에서 S3 업로드

Transfer Acceleration은 가까운 엣지 로케이션에서 AWS 백본으로 데이터를 실어 **먼 거리 업로드**를 빠르게 한다. 멀티파트 업로드는 큰 파일을 나눠 **병렬 전송**하고 실패한 부분만 다시 보낸다(100MB 이상이면 권장).

📎 [S3 Transfer Acceleration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html) · [멀티파트 업로드](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)

**Q57 — 정답 D** · 복원력 · 다중 리전 활성-활성 NoSQL

DynamoDB 글로벌 테이블은 여러 리전에 **다중 활성(multi-active)** 복제본을 두어 각 리전에서 읽기·쓰기가 된다. Aurora Global Database는 쓰기 리전이 하나다(보조 리전은 읽기, 쓰기 전달 기능은 별도). DAX는 같은 리전 캐시다.

📎 [DynamoDB 글로벌 테이블](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)

**Q58 — 정답 D** · 고성능 · UDP 글로벌 성능

Global Accelerator는 UDP·TCP 트래픽을 가까운 엣지에서 AWS 백본으로 보내고 **애니캐스트 고정 IP**를 준다. CloudFront는 HTTP(S) 기반이고, ALB는 UDP를 지원하지 않는다.

📎 [AWS Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)

**Q59 — 정답 C** · 복원력 · DNS 장애 조치

장애 조치 라우팅은 기본 레코드의 **상태 확인이 실패하면** 보조 레코드로 응답한다. 가중치 라우팅은 정상 상황에서도 트래픽을 나눈다. ALB는 S3를 대상으로 쓸 수 없다.

📎 [Route 53 장애 조치 라우팅](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-failover.html)

**Q60 — 정답 D** · 보안 · S3 민감 데이터 발견

Macie는 머신러닝·패턴 매칭으로 S3의 **민감 데이터**를 찾아 내고 버킷의 보안 상태도 평가한다. Inspector는 소프트웨어 취약점 스캔, Config는 설정 규칙 준수, Detective는 보안 사고 조사 도구다.

📎 [Amazon Macie](https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html)

</details>

<details>
<summary><b>Q61 ~ Q65 해설</b></summary>

**Q61 — 정답 D,E** · 복원력 · 객체 리전 간 복사

S3 복제는 **원본과 대상 모두 버전 관리가 켜져 있어야** 하고, 복제 규칙과 S3가 맡을 IAM 역할이 필요하다. 수명 주기는 스토리지 클래스 전환·만료이지 리전 이동이 아니다. 규칙을 만들기 **전**부터 있던 객체는 S3 Batch Replication으로 따로 복제한다.

📎 [S3 복제](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

**Q62 — 정답 C** · 복원력 · 단일 인스턴스 자동 복구

용량 1인 Auto Scaling 그룹은 인스턴스가 비정상이 되면 **자동으로 교체**하고, 여러 AZ를 지정하면 AZ 장애 때 다른 AZ에 띄운다. 데이터는 EBS가 아닌 EFS·S3·RDS 같은 외부에 두어야 한다.

📎 [Auto Scaling 상태 확인](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html)

**Q63 — 정답 D** · 비용 최적화 · NAT 처리 요금

DynamoDB도 S3처럼 **무료 게이트웨이 엔드포인트**를 지원한다. NAT를 우회해 처리 요금이 사라진다. 인터페이스 엔드포인트는 시간·GB 요금이 있다.

📎 [게이트웨이 엔드포인트](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html)

**Q64 — 정답 B** · 복원력 · 실패 메시지 격리

DLQ는 정해진 횟수 이상 받은(실패한) 메시지를 **따로 격리**해 원인 분석·재처리를 할 수 있게 한다. 보존 기간을 줄이면 정상 메시지도 잃는다.

📎 [SQS DLQ](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)

**Q65 — 정답 D** · 복원력 · 여러 단계 워크플로

Step Functions는 단계·분기·재시도·대기를 **상태 머신**으로 관리하고, Standard 워크플로는 최대 1년까지 실행되며 **작업 토큰**으로 사람 승인 같은 외부 콜백을 기다릴 수 있다. Lambda 하나는 최대 실행 시간이 15분이다.

📎 [Step Functions 작업 토큰](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html#connect-wait-token)

</details>


---

## 7. 채점과 결과 해석

### 7-1. 도메인별 점수표

정답표를 보며 도메인별로 맞힌 개수를 세어 채우세요. 문항 번호는 이 모의고사의 배치 순서입니다.

| 도메인 | 문항 번호 | 문항 수 | 내 정답 수 | 정답률 |
|---|---|---|---|---|
| D1 안전한 아키텍처 (30%) | 4, 6, 9, 10, 13, 15, 17, 18, 20, 23, 26, 29, 34, 35, 37, 39, 40, 41, 54, 60 | 20 | ___ | ___ % |
| D2 복원력 있는 아키텍처 (26%) | 2, 5, 7, 8, 19, 25, 30, 43, 44, 48, 55, 57, 59, 61, 62, 64, 65 | 17 | ___ | ___ % |
| D3 고성능 아키텍처 (24%) | 12, 14, 21, 27, 28, 33, 36, 38, 45, 46, 50, 51, 52, 56, 58 | 15 | ___ | ___ % |
| D4 비용 최적화 아키텍처 (20%) | 1, 3, 11, 16, 22, 24, 31, 32, 42, 47, 49, 53, 63 | 13 | ___ | ___ % |
| **합계** | | **65** | ___ | ___ % |

### 7-2. 점수대별 다음 행동

| 맞힌 개수(65문항) | 해석 | 다음 행동 |
|---|---|---|
| 56 이상 (86%+) | 기본기 충분 | 바로 [W07 모의고사 2 (함정 난이도)](./w07-mock-exam-2.md)로 |
| 52~55 (80~85%) | 안전선 통과 | 틀린 문항의 도메인 편만 복습하고 W07로 |
| 47~51 (72~79%) | 참고선 근처, 운에 따라 갈림 | 정답률이 가장 낮은 도메인 편을 다시 읽고, 1주 뒤 이 모의고사를 **처음부터** 다시 |
| 46 이하 | 도메인 하나 이상이 비어 있음 | 아래 표에서 약한 도메인 편부터 다시. W05 치트시트의 "대표 단서"를 외우는 것부터 |

### 7-3. 약한 도메인 → 다시 볼 편

| 정답률이 낮은 도메인 | 다시 볼 편 | 특히 볼 것 |
|---|---|---|
| D1 보안 | [W01 안전한 아키텍처](./w01-secure-architectures.md) | IAM 역할·SCP, KMS vs CloudHSM, 엔드포인트, WAF·Shield·GuardDuty·Macie |
| D2 복원력 | [W02 복원력 있는 아키텍처](./w02-resilient-architectures.md) | Multi-AZ vs 복제본, DR 4단계, SQS·SNS 분리, Route 53 장애 조치 |
| D3 고성능 | [W03 고성능 아키텍처](./w03-high-performing-architectures.md) | 캐시(CloudFront·DAX·ElastiCache), 배치 그룹, EBS 유형, Kinesis |
| D4 비용 | [W04 비용 최적화](./w04-cost-optimization.md) | 구매 옵션, S3 계층, NAT 함정, Budgets·Compute Optimizer |
| 여러 도메인에서 "서비스 고르기" 실수 | [W05 서비스 비교 치트시트](./w05-service-comparison.md) | 헷갈리는 쌍 표 |

> 💡 **손으로 확인하면 오래 갑니다.** 틀린 문항의 서비스를 실제로 만들어 보는 실습은 클라우드 탭에서 할 수 있습니다: VPC·엔드포인트·NAT는 [cloud02](../../cloud/cloud02-vpc-networking.md), EC2·Auto Scaling·ALB는 [cloud03](../../cloud/cloud03-ec2-asg-alb.md), S3·RDS는 [cloud04](../../cloud/cloud04-s3-rds.md), 보안 서비스는 [cloud09](../../cloud/cloud09-security.md), 비용 도구는 [cloud11](../../cloud/cloud11-finops-cleanup.md).

---

## 자주 하는 실수

| 실수 | 왜 틀리나 | 고치는 법 |
|---|---|---|
| "가장 비용 효율적인"만 보고 가장 싼 보기 선택 | Spot·One Zone·단일 AZ가 **가용성·내구성 조건**을 어김 | 조건을 모두 만족하는 보기 중에서만 비교 |
| 읽기 전용 복제본을 고가용성 답으로 | 비동기·수동 승격, 자동 장애 조치 아님 | "자동 장애 조치" → Multi-AZ |
| Multi-AZ 대기 인스턴스로 읽기 분산 | Multi-AZ DB 인스턴스의 대기는 읽기를 못 받음 | 읽기 → 복제본 |
| "고정 IP"·"UDP"를 놓치고 CloudFront | CloudFront는 HTTP(S) 캐시, 고정 IP 없음 | → Global Accelerator |
| 보안 그룹으로 IP 거부 | SG는 허용 규칙만 | 거부 → NACL(또는 WAF IP 집합) |
| IAM 정책으로 조직 전체 강제 | 계정 관리자가 바꿀 수 있음 | 조직 강제 → SCP |
| 복수 선택에서 하나만 확신하고 아무거나 | 하나라도 틀리면 0점 | 두 보기가 **서로 다른 요구**를 하나씩 채우는지 확인 |

---

## 핵심 정리

- 실제 SAA-C03은 **65문항(채점 50 + 비채점 15)·130분, 100~1,000 척도·720점 합격, 보상 모델, 감점 없음**이다. 모르는 문항도 반드시 찍는다.
- 문항은 **질문 문장 → 조건 단어 → 조건 위반 보기 제거 → 꼬리표로 선택** 순서로 푼다.
- 기본 난이도 문항의 대부분은 **"서비스의 대표 용도"** 를 정확히 알면 풀린다: 역할(EC2 권한), SCP(조직 강제), OAC(S3 직접 접근 차단), Multi-AZ(자동 장애 조치), SQS(급증 흡수), SNS→SQS(팬아웃), DAX(µs 읽기), Global Accelerator(UDP·고정 IP), Compute SP(변경 예정 약정), 게이트웨이 엔드포인트(NAT 비용).
- 도메인별 점수표에서 **가장 낮은 도메인 하나**만 골라 해당 편을 다시 보는 것이 가장 효율적인 복습이다.

---

## ⏭️ 다음 예고 — AWS SAA W07. 모의고사 2 (함정 난이도)

W07도 65문항·130분이지만 결이 다릅니다. **보기 네 개가 모두 동작하는** 문항이 대부분이고, 그중 "운영 부담이 가장 적은", "가장 비용 효율적인", "코드 변경을 최소화하는" 같은 꼬리표 하나로 답이 갈립니다. NAT 하나를 공유할지 AZ마다 둘지처럼 **비용과 복원력이 부딪치는** 문제, 128KB보다 작은 객체 수억 개를 IA로 옮기면 오히려 비싸지는 문제, Intelligent-Tiering·Database Savings Plans·EBS Multi-Attach처럼 **정답처럼 보이지만 조건 하나를 어기는** 보기가 기다립니다. 복수 선택 문항도 늘어납니다. W06에서 80%를 넘겼다면 도전해 보세요.
