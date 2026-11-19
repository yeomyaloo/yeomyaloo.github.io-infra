---
title: "AWS SAA W07. 모의고사 2 — 함정 난이도 (65문항 · 130분)"
date: 2026-11-19T22:35:17+09:00
weight: 8
tags: ["aws-saa", "certification", "aws", "mock-exam"]
summary: "[직접 만든 창작 문항 · 기출·덤프 아님] 보기가 모두 동작하는 문항에서 꼬리표(\"운영 부담이 가장 적은\", \"가장 비용 효율적인\", \"코드 변경 최소\", \"추가 요금 없이\")로 답을 가른다. 비용과 복원력·성능이 부딪치는 트레이드오프 문항에서 요구 사항의 우선순위를 읽어 낸다."
image: ""
draft: false
---
> **자격증 준비 · AWS SAA** | 난이도 ★★★★☆ | 예상 소요 시험 130분 + 채점·복기 2~2.5시간

> ⚠️ **이 모의고사의 모든 문항은 AWS가 공개한 SAA-C03 시험 가이드(2026-11-19 확인, 가이드 버전 1.1)의 4개 도메인과 과제 설명을 바탕으로 이 시리즈가 직접 창작한 연습 문제입니다.** 실제 시험 문제나 유출 문제(덤프)가 아니고, 그런 것을 재현하지도 않습니다. AWS 자격증 시험은 응시 계약(NDA)으로 보호되므로 "실제 출제 문제"라고 주장하는 자료는 믿지도, 퍼뜨리지도 마세요. 회사 이름·수치·시나리오는 모두 새로 지은 것입니다.

## 🎯 오늘의 목표

- **보기가 모두 동작하는** 문항에서 꼬리표("운영 부담이 가장 적은", "가장 비용 효율적인", "코드 변경 최소", "추가 요금 없이")로 답을 가른다
- **비용과 복원력·성능이 부딪치는** 트레이드오프 문항에서 요구 사항의 우선순위를 읽어 낸다
- "정답처럼 보이지만 **조건 하나를 어기는** 보기"(Express 워크플로, Memcached, One Zone, 128KB 미만 객체, Compute SP와 RDS 등)를 걸러 낸다
- 정답표·해설·**도메인별 점수표**로 W06과 비교해 실전 준비 상태를 판단한다

---

## 1. 시험 안내

### 1-1. 형식과 규칙

W06과 같은 형식입니다. 실제 시험 정보(2026-11-19 AWS 공식 페이지·시험 가이드 확인)를 다시 정리합니다.

| 항목 | 이 모의고사 | 실제 시험 SAA-C03 |
|---|---|---|
| 문항 수 | **65문항**, 모두 채점 | 65문항 = 채점 50 + 비채점 15(표시 없음) |
| 시간 | **130분** | 130분 |
| 형식 | 단일 선택(보기 4개) 58문항 + 복수 선택(보기 5개 중 2개) 7문항 | 단일 선택 + 복수 선택 |
| 채점 | 맞힌 개수 → 도메인별 정답률 | 100~1,000 척도 점수, **720점 합격**, 보상 모델, 감점 없음 |
| 합격 참고선 | **47문항(72%)**, 안전선 52문항(80%) | 척도 점수는 정답률과 1:1 대응하지 않음 |
| 도메인 비중 | D1 20 · D2 17 · D3 15 · D4 13문항 | D1 30% · D2 26% · D3 24% · D4 20% |

> 💡 **이 모의고사는 일부러 어렵게 만들었습니다.** W06보다 점수가 5~10문항 낮게 나오는 것이 정상입니다. 실제 시험에는 이런 함정 문항과 W06 같은 기본 문항이 섞여 나오므로, W06 80% + W07 72% 이상이면 실전 준비가 된 것으로 보아도 좋습니다.

### 1-2. 진행 방법

1. 타이머 **130분**. 답은 `Q1 B`, `Q2 A,D`처럼 적고, 헷갈리면 `?` 표시.
2. 복수 선택은 "(2개 선택)"이 문제에 적혀 있고 **둘 다 맞아야** 정답입니다.
3. 1차 풀이 100분, 검토 30분을 권장합니다. 함정 문항은 오래 붙잡지 말고 **`?` 표시 후 넘어가세요.**

---

## 2. 이번 회의 함정 유형

문항을 풀기 전에 한 번 읽고, 풀면서 "지금 이 유형인가?"를 떠올리세요.

| 유형 | 모습 | 대응 |
|---|---|---|
| ① **모두 동작함** | 보기 네 개가 전부 요구를 만족 | 꼬리표 단어(운영 부담·비용·코드 변경·추가 요금)로만 판정 |
| ② **과한 정답** | 요구보다 훨씬 강한 해법(Global Database, io2, 다중 사이트) | "가장 비용 효율적"이면 **요구를 딱 만족하는** 가장 싼 해법 |
| ③ **이름 함정** | Express 워크플로(최대 5분), Memcached(복제 없음), One Zone(AZ 하나) | 이름이 아니라 **한도·범위**로 확인 |
| ④ **범위 함정** | 기본 암호화(리전 단위), CloudTrail 기본(관리 이벤트만), CRR(이후 객체만) | "기존 것", "모든 리전", "객체 단위" 같은 단어에 밑줄 |
| ⑤ **비용 ↔ 복원력** | NAT 하나 vs AZ별, One Zone vs 리전, 단일 DX vs 이중화 | **환경(개발/운영)과 허용 중단**을 먼저 읽기 |
| ⑥ **할인 적용 범위** | Compute SP는 RDS에 미적용, IA는 128KB 최소 과금, Intelligent-Tiering은 작은 객체 미이동 | 할인·계층 규칙의 **예외**를 기억 |
| ⑦ **탐지 vs 예방** | Config·GuardDuty·Access Analyzer(탐지) vs SCP·버킷 정책(예방) | "막아야", "누구도 못 하게" → 예방 수단 |

---

## 3. 문항

> ⏱️ **지금 타이머를 시작하세요.** 130분.

**Q1.** S3 리전 간 복제(CRR) 규칙을 오늘 설정했다. 그런데 **규칙을 만들기 전부터 있던** 객체 3억 개는 대상 버킷에 나타나지 않는다. 가장 알맞은 방법은?

- A. DataSync로 두 버킷을 매일 동기화한다
- B. 수명 주기 규칙으로 객체를 대상 리전으로 전환한다
- C. S3 Batch Replication 작업으로 기존 객체를 복제한다
- D. CRR 규칙을 지웠다가 다시 만든다

**Q2.** 온프레미스 서버가 VPC의 **Route 53 프라이빗 호스팅 영역** 이름(`db.internal.example`)을 조회해야 하고, VPC의 인스턴스는 온프레미스 DNS 영역(`corp.example`)을 조회해야 한다. Direct Connect로 연결되어 있다. 필요한 구성을 **2개** 고르시오.

- A. 프라이빗 호스팅 영역을 퍼블릭 호스팅 영역으로 바꾼다
- B. Route 53 Resolver **인바운드 엔드포인트**를 만들고 온프레미스 DNS가 `db.internal.example` 질의를 그 IP로 전달하게 한다
- C. 온프레미스 DNS에 VPC의 모든 레코드를 수동으로 복사한다
- D. VPC의 DHCP 옵션에서 온프레미스 DNS만 쓰게 한다
- E. Route 53 Resolver **아웃바운드 엔드포인트**와 `corp.example`용 **전달 규칙**을 만들어 VPC에 연결한다

**Q3.** 규정상 TLS를 **애플리케이션 서버에서만** 종료해야 하고(로드 밸런서에서 복호화 금지), 클라이언트 원본 IP가 서버에 보여야 하며, 연결 수가 매우 많다. 알맞은 로드 밸런서 구성은?

- A. NLB에 **TCP 리스너(443)** 를 만들어 TLS를 그대로 대상에 전달한다
- B. ALB에 HTTPS 리스너를 만들고 대상도 HTTPS로 한다
- C. Gateway Load Balancer를 쓴다
- D. NLB에 TLS 리스너를 만든다

**Q4.** 개발 VPC는 2개 AZ에 프라이빗 서브넷이 있고, 인터넷 트래픽은 하루 수 GB다. 개발 환경은 **AZ 장애 시 몇 시간 중단돼도 괜찮다**. 지금은 AZ마다 NAT 게이트웨이가 있다. **가장 비용 효율적인** 구성은?

- A. NAT 게이트웨이를 하나로 줄이고 두 AZ 프라이빗 서브넷이 그것을 함께 쓴다
- B. Transit Gateway로 운영 VPC의 NAT를 공유한다
- C. AZ마다 NAT 게이트웨이를 유지한다
- D. NAT 게이트웨이를 없애고 모든 인스턴스에 퍼블릭 IP를 붙인다

**Q5.** 같은 리전의 VPC 3개가 서로 대량의 데이터를 주고받는다(월 수십 TB). VPC를 더 늘릴 계획은 없고, CIDR은 겹치지 않는다. **데이터 전송 비용이 가장 적은** 연결 방법은?

- A. VPC 세 쌍을 **VPC 피어링**으로 풀 메시 연결한다
- B. Site-to-Site VPN으로 서로 연결한다
- C. 각 VPC에 서로의 PrivateLink 엔드포인트 서비스를 만든다
- D. Transit Gateway에 세 VPC를 연결한다

**Q6.** 회사는 RDS for MySQL(db.r7g.2xlarge, 서울, Multi-AZ)을 **앞으로 3년 동안** 같은 구성으로 쓸 것이 확실하다. 엔진·인스턴스 변경 계획은 없다. **할인 폭이 가장 큰** 방법은?

- A. 1년 Database Savings Plans
- B. 3년 Compute Savings Plans
- C. 온디맨드를 유지하고 밤에 중지
- D. 3년 **RDS 예약 인스턴스**(Multi-AZ, 전액 선결제)

**Q7.** 규정상 **앞으로 만드는 모든 EBS 볼륨과 스냅숏**이 암호화되어야 한다. 개발자가 옵션을 빼먹어도 지켜져야 한다. 가장 운영 부담이 적은 방법은?

- A. 모든 AMI를 암호화해서 다시 만든다
- B. IAM 정책으로 `ec2:CreateVolume`을 막는다
- C. 리전마다 EC2의 **EBS 기본 암호화(encryption by default)** 를 켠다
- D. AWS Config 규칙으로 암호화되지 않은 볼륨을 찾아 알린다

**Q8.** Java 21 Lambda API의 콜드 스타트가 6초 걸린다. 트래픽은 하루 종일 들쭉날쭉하다. **추가 요금 없이** 콜드 스타트를 크게 줄이는 방법은?

- A. Lambda **SnapStart**를 켜고 게시된 버전(또는 별칭)으로 호출한다
- B. 함수를 VPC에서 분리한다
- C. 메모리를 10GB로 늘린다
- D. 프로비저닝된 동시성을 50으로 설정한다

**Q9.** 감사 결과, 일부 클라이언트가 S3 버킷에 **HTTP(암호화되지 않은 연결)** 로 요청하는 것으로 나왔다. 버킷 수준에서 암호화되지 않은 요청을 모두 거부하려면?

- A. 버킷 정책에 `aws:SecureTransport`가 `false`인 요청을 거부(Deny)하는 문장을 추가한다
- B. 버킷 기본 암호화를 SSE-KMS로 바꾼다
- C. S3 퍼블릭 액세스 차단을 켠다
- D. 버킷 ACL을 비공개로 바꾼다

**Q10.** SaaS 모니터링 업체가 고객 계정에 IAM 역할을 만들어 달라고 한다. 업체는 수천 고객의 역할을 같은 AWS 계정에서 맡는다. 다른 고객이 업체를 속여 **우리 계정 역할을 맡게 하는 문제(혼동된 대리자)** 를 막으려면?

- A. 업체 계정에 MFA를 요구한다
- B. 역할의 최대 세션 시간을 15분으로 줄인다
- C. 역할 이름을 추측하기 어렵게 만든다
- D. 역할 신뢰 정책에 업체 계정을 주체로 넣고, 업체가 고객별로 발급한 **외부 ID(`sts:ExternalId`)** 조건을 건다

**Q11.** 대출 심사 워크플로는 자동 심사 후 **심사역 승인을 최대 5일** 기다린 다음 계약을 진행한다. 워크플로 실행 이력을 감사용으로 남겨야 한다. 알맞은 선택은?

- A. Step Functions **Express** 워크플로 + 대기 상태
- B. Step Functions **Standard** 워크플로 + `.waitForTaskToken` 콜백
- C. SQS 지연 큐로 5일 뒤 다시 처리
- D. Lambda 함수에서 5일 동안 폴링

**Q12.** 쇼핑몰은 로그인 세션을 ElastiCache에 저장한다. 캐시 노드 하나가 장애 나도 **세션을 잃지 않고 자동으로** 서비스가 이어져야 한다. 알맞은 구성은?

- A. ElastiCache Memcached 노드를 여러 AZ에 3개 둔다
- B. 캐시 대신 EC2 로컬 디스크에 세션을 저장한다
- C. ElastiCache(Valkey 또는 Redis OSS)에 **복제본을 두고 Multi-AZ 자동 장애 조치**를 켠다
- D. Memcached 노드를 더 큰 유형으로 바꾼다

**Q13.** 보안팀은 계정이 탈취되어도 공격자가 **백업을 지우거나 보존 기간을 줄이지 못하게** 하려 한다. 필요한 조치를 **2개** 고르시오.

- A. 같은 계정 볼트에 백업하고 IAM 정책으로 삭제를 거부한다
- B. AWS Backup 볼트에 **Vault Lock(규정 준수 모드)** 을 건다
- C. EBS 스냅숏을 퍼블릭으로 공유해 둔다
- D. 백업을 **다른 계정(가능하면 다른 리전)의 볼트로 복사**한다
- E. 백업 보존 기간을 10년으로 늘린다

**Q14.** 여러 VPC의 워크로드가 인터넷으로 나갈 때 **허용된 도메인(예: `*.github.com`, 패키지 저장소)으로만** 나가게 하고, 그 밖의 도메인으로 가는 HTTPS 연결은 차단하려 한다. 중앙에서 관리할 수 있어야 한다. 알맞은 서비스는?

- A. AWS Network Firewall의 상태 저장 도메인 목록 규칙(TLS SNI·HTTP Host 기반)을 검사 VPC에 두고 Transit Gateway로 트래픽을 보낸다
- B. 보안 그룹의 아웃바운드 규칙에 도메인 이름을 넣는다
- C. 네트워크 ACL에서 도메인 이름을 거부한다
- D. AWS WAF를 NAT 게이트웨이에 연결한다

**Q15.** 보안팀은 Organizations의 **모든 멤버 계정에서**, 각 계정의 관리자(AdministratorAccess 보유)를 포함해 누구도 조직 CloudTrail 추적을 중지·삭제하지 못하게 하려 한다. 가장 확실한 방법은?

- A. 각 계정 관리자 그룹에 CloudTrail 변경을 거부하는 IAM 정책을 붙인다
- B. CloudTrail 로그 버킷에 Object Lock을 건다
- C. AWS Config 규칙 `cloudtrail-enabled`로 꺼지면 알림을 보낸다
- D. 관리 계정에서 조직 추적(organization trail)을 만들고, `cloudtrail:StopLogging`·`cloudtrail:DeleteTrail`을 거부하는 SCP를 멤버 계정 OU에 연결한다

**Q16.** 사내 전용 REST API(API Gateway)를 **VPC 안의 클라이언트만** 호출할 수 있게 하고, 인터넷에서는 아예 보이지 않게 하려 한다. 알맞은 구성은?

- A. API를 **프라이빗 REST API**로 만들고, VPC에 `execute-api` 인터페이스 엔드포인트를 둔 뒤 리소스 정책으로 그 엔드포인트만 허용한다
- B. 리전 REST API를 만들고 WAF에서 VPC CIDR만 허용한다
- C. HTTP API를 만들고 Lambda 권한 부여자로 IP를 검사한다
- D. 엣지 최적화 API를 만들고 API 키를 요구한다

**Q17.** 주문 이벤트를 SNS 토픽으로 발행한다. 배송 큐는 `type=SHIPPED`, 환불 큐는 `type=REFUND` 메시지만 받아야 하고, 감사 큐는 모든 메시지를 받아야 한다. **코드를 추가하지 않고** 구현하려면?

- A. 세 큐가 모든 메시지를 받고 소비자 코드에서 버린다
- B. 이벤트 유형마다 SNS 토픽을 따로 만들고 발행자가 골라 보낸다
- C. 각 SQS 구독에 **구독 필터 정책**을 설정한다(감사 큐는 필터 없음)
- D. Lambda로 메시지를 받아 유형별 큐로 보낸다

**Q18.** 로그 수집기가 초당 2만 건의 `PUT`을 하나의 S3 접두사(`logs/`)에 보내자 `503 Slow Down` 오류가 자주 난다. 가장 알맞은 조치는?

- A. 버전 관리를 끈다
- B. S3 Standard-IA로 저장한다
- C. 버킷에 Transfer Acceleration을 켠다
- D. 객체 키를 여러 접두사(예: `logs/<해시 일부>/...` 또는 날짜·시간 분할)로 나눠 요청을 분산한다

**Q19.** 투표 서비스가 DynamoDB 테이블에 `candidate_id`를 파티션 키로 쓰는데, 인기 후보 한 명에게 쓰기가 몰려 **스로틀링**이 난다. 테이블 전체 용량은 넉넉하다. 가장 알맞은 해결책은?

- A. 온디맨드 모드로 바꾸기만 한다
- B. DAX를 추가한다
- C. 쓰기 샤딩: `candidate_id#<0~N 난수>`처럼 접미사를 붙여 쓰기를 여러 파티션으로 분산하고, 읽을 때 합산한다
- D. 글로벌 보조 인덱스를 추가한다

**Q20.** Aurora PostgreSQL의 읽기 트래픽이 시간대마다 크게 출렁인다. 읽기 지연을 일정하게 유지하면서 **읽기 용량이 자동으로** 늘고 줄게 하려면? 애플리케이션은 읽기 연결을 한 곳으로만 보낼 수 있다.

- A. Aurora Auto Scaling으로 Aurora 복제본 수를 CPU·연결 수 기준으로 조절하고, 애플리케이션은 **리더 엔드포인트**로 읽는다
- B. 복제본을 15개 만들어 두고 인스턴스 엔드포인트를 애플리케이션이 고르게 한다
- C. Aurora Global Database를 구성한다
- D. writer 인스턴스 크기를 키운다

**Q21.** 게임 서버는 수백만 플레이어의 점수로 **실시간 상위 100명 순위표**를 초당 수만 번 조회한다. 지금은 RDS에 `ORDER BY score DESC LIMIT 100`을 날려 DB가 버티지 못한다. 가장 알맞은 방법은?

- A. DynamoDB로 옮기고 Scan으로 정렬한다
- B. CloudFront로 쿼리 결과를 캐시한다
- C. RDS 읽기 전용 복제본을 5개 추가한다
- D. ElastiCache(Valkey/Redis OSS)의 **정렬된 집합(sorted set)** 으로 순위를 관리한다

**Q22.** Auto Scaling 그룹의 인스턴스 하나가 애플리케이션 오류로 HTTP 500만 반환한다. ALB는 이 인스턴스를 비정상으로 표시해 트래픽을 보내지 않지만, Auto Scaling 그룹은 인스턴스를 **교체하지 않는다**. 원인과 해결로 옳은 것은?

- A. 인스턴스의 세부 모니터링을 켜야 한다
- B. ALB 대상 그룹의 상태 확인 경로를 지워야 한다
- C. 그룹의 상태 확인 유형이 EC2(상태 검사)만이라서다. **ELB 상태 확인**을 켜야 교체된다
- D. 축소 보호(scale-in protection)를 켜야 한다

**Q23.** 회사는 50개 계정에서 GuardDuty·Inspector·Macie·Config를 쓴다. 보안팀은 **한 화면에서** 모든 계정의 결과를 보고 우선순위를 매기며, 보안 모범 사례 점검 점수도 보고 싶다. 알맞은 방법은?

- A. 보안 계정을 AWS Security Hub 위임 관리자로 지정하고 조직 전체에 Security Hub를 켠다
- B. Amazon Detective만 켠다
- C. 각 계정의 GuardDuty 콘솔을 차례로 확인한다
- D. 모든 결과를 CloudWatch Logs로 보내 직접 대시보드를 만든다

**Q24.** S3 이벤트로 비동기 호출되는 Lambda 함수가 재시도 후에도 실패하면, 원래 이벤트와 함께 **오류 메시지·스택 정보 같은 실패 맥락**을 SQS로 보내 나중에 분석하려 한다. 가장 알맞은 설정은?

- A. 비동기 호출 설정에서 **실패 시 대상(on-failure destination)** 으로 SQS 큐를 지정한다
- B. 함수에 배달 못한 편지 큐(DLQ)를 지정한다
- C. 예약된 동시성을 0으로 설정한다
- D. S3 이벤트 알림을 SQS로 바꾼다

**Q25.** CloudFront 배포의 캐시 적중률이 5%에 불과해 오리진 부하가 크다. 확인해 보니 캐시 키에 **모든 쿼리 문자열과 모든 쿠키**가 포함되어 있는데, 실제로 응답을 바꾸는 것은 `lang` 쿼리 문자열 하나뿐이다. 가장 알맞은 조치는?

- A. TTL을 0으로 줄인다
- B. 오리진 인스턴스를 두 배로 늘린다
- C. 캐시 정책에서 캐시 키에 `lang` 쿼리 문자열만 포함하고 쿠키는 제외한다(오리진에 꼭 필요한 값은 오리진 요청 정책으로 전달)
- D. Origin Shield를 끈다

**Q26.** VPC의 프라이빗 서브넷에 연결된 Lambda 함수가 같은 리전의 S3 버킷과 DynamoDB 테이블을 읽고 쓴다. VPC에는 인터넷 게이트웨이와 NAT가 없고, 앞으로도 만들지 않는다. **추가 비용이 가장 적게** 드는 방법을 **2개** 고르시오.

- A. S3와 DynamoDB용 인터페이스 엔드포인트를 AZ마다 만든다
- B. DynamoDB 게이트웨이 엔드포인트를 만들고 Lambda 서브넷의 라우팅 테이블에 연결한다
- C. Lambda를 VPC에서 분리한다
- D. NAT 게이트웨이를 하나 만들고 사용 후 지운다
- E. S3 게이트웨이 엔드포인트를 만들고 Lambda 서브넷의 라우팅 테이블에 연결한다

**Q27.** 회사의 EC2 사용은 세 부분이다. ① 24시간 꾸준한 웹 계층(인스턴스 패밀리는 앞으로 바뀔 수 있음), ② 낮 시간 예측 불가 급증, ③ 밤마다 재시도 가능한 배치. **가장 비용 효율적인** 조합을 **2개** 고르시오(급증분은 온디맨드로 처리한다고 가정).

- A. ① ② ③ 최대 사용량만큼 3년 Standard RI
- B. ③ 배치는 여러 인스턴스 유형을 허용한 **Spot**
- C. ② 급증분까지 포함해 Savings Plans를 최대 사용량만큼 약정
- D. ③ 배치는 온디맨드 용량 예약
- E. ① 꾸준한 기준 사용량만큼 **Compute Savings Plans**

**Q28.** 교육 회사가 S3의 수 GB 강의 파일을 **결제한 회원에게만** 일정 시간 동안 내려받게 하려 한다. 회원은 전 세계에 있고, 인기 강의는 같은 파일이 수천 번 다운로드된다. 지연과 전송 비용도 줄이고 싶다. 가장 알맞은 방법은?

- A. CloudFront(OAC로 S3 접근 제한)를 두고 CloudFront **서명된 URL**을 회원에게 발급한다
- B. S3 버킷을 퍼블릭으로 열고 URL을 추측하기 어렵게 만든다
- C. Global Accelerator 뒤에 S3를 둔다
- D. S3 미리 서명된(presigned) URL을 회원에게 발급한다

**Q29.** 200GiB 데이터를 쓰는 DB 서버가 **일정하게 10,000 IOPS**를 필요로 한다. 지금은 gp2 볼륨을 3,400GiB로 키워 IOPS를 맞추고 있다. 성능을 유지하며 **가장 비용 효율적인** 방법은?

- A. gp2 볼륨을 그대로 두고 RAID 0으로 묶는다
- B. io2 볼륨 200GiB에 10,000 IOPS를 프로비저닝한다
- C. 200GiB 정도의 **gp3** 볼륨에 10,000 IOPS를 따로 프로비저닝한다
- D. st1 볼륨으로 바꾼다

**Q30.** Lambda 함수 20개가 외부 API 키 한 개(교체 요구 없음)와 기능 플래그 30개를 읽는다. 값은 암호화되어 있어야 하고, **비용이 가장 적어야** 한다. 알맞은 방법은?

- A. 모든 값을 Secrets Manager 비밀 31개로 저장한다
- B. Systems Manager Parameter Store 표준 파라미터에 저장하고 API 키는 SecureString(KMS)으로 둔다
- C. S3에 JSON으로 저장하고 퍼블릭 읽기를 허용한다
- D. Lambda 환경 변수에 평문으로 넣는다

**Q31.** ALB 뒤 애플리케이션 서버는 Auto Scaling으로 IP가 계속 바뀐다. 애플리케이션 서버의 8080 포트는 **ALB에서 오는 트래픽만** 받아야 한다. 가장 관리하기 쉬운 규칙은?

- A. 애플리케이션 서버 보안 그룹에 VPC CIDR 전체를 허용한다
- B. 네트워크 ACL에서 ALB 서브넷만 허용하고 보안 그룹은 0.0.0.0/0으로 둔다
- C. ALB의 현재 프라이빗 IP들을 /32로 하나씩 등록한다
- D. 애플리케이션 서버 보안 그룹의 인바운드 규칙 출발지로 **ALB의 보안 그룹 ID**를 지정한다

**Q32.** 주문 시스템의 DR 요구는 **RTO 10분**이다. 평소에도 보조 리전에서 **축소된 규모로 실제 트래픽을 처리할 수 있는** 환경을 유지할 예산은 있지만, 주 리전과 같은 규모를 항상 켜 둘 예산은 없다. 알맞은 전략은?

- A. 다중 사이트 활성-활성(multi-site active/active)
- B. 웜 스탠바이(warm standby)
- C. 백업·복원(backup and restore)
- D. 파일럿 라이트(pilot light)

**Q33.** 클러스터 인식 파일 시스템(GFS2 등)을 쓰는 레거시 고가용성 애플리케이션이 **같은 AZ의 EC2 두 대**에서 **하나의 블록 디바이스를 동시에** 읽고 써야 한다. 알맞은 방법은?

- A. gp3 볼륨을 두 인스턴스에 연결한다
- B. EFS를 블록 디바이스로 마운트한다
- C. 인스턴스 스토어를 공유한다
- D. io2 EBS 볼륨에 **Multi-Attach**를 켜고 두 Nitro 인스턴스에 연결한다

**Q34.** B2B 결제 API(HTTPS, 동적 응답, 캐시 불가)는 두 리전의 ALB에서 운영된다. 고객사 방화벽은 **IP 허용 목록**만 지원하고, 리전 장애 시 **1분 안에** 다른 리전으로 넘어가야 한다. 가장 알맞은 방법은?

- A. Route 53 장애 조치 라우팅만 쓴다
- B. CloudFront 배포를 두고 오리진 페일오버를 설정한다
- C. AWS Global Accelerator를 두고 두 리전의 ALB를 엔드포인트 그룹으로 등록한다
- D. 두 ALB의 IP를 고객사에 알려 준다

**Q35.** 주 리전(서울)의 ALB와 보조 리전(도쿄)의 ALB가 있다. 서울 애플리케이션이 비정상이면 **자동으로** 도쿄로 DNS를 돌리고 싶다. 필요한 설정을 **2개** 고르시오.

- A. 기본 레코드(ALB 별칭)에서 **대상 상태 평가(Evaluate target health)** 를 켜거나 Route 53 상태 확인을 연결한다
- B. Route 53 Resolver 인바운드 엔드포인트를 만든다
- C. 레코드 TTL을 0으로 만든다
- D. 두 레코드를 가중치 50:50으로 만든다
- E. 서울·도쿄 레코드를 **장애 조치 라우팅 정책**의 기본(primary)·보조(secondary)로 만든다

**Q36.** Aurora MySQL 클러스터의 장애 조치는 30초 안에 끝나지만, 애플리케이션은 DNS 캐시와 끊긴 연결 때문에 **수 분 동안 오류**를 낸다. 애플리케이션 코드 변경을 최소화하며 이 시간을 줄이려면?

- A. Aurora 복제본을 15개로 늘린다
- B. 클러스터 엔드포인트 대신 인스턴스 엔드포인트를 쓴다
- C. RDS Proxy를 두고 애플리케이션이 프록시 엔드포인트에 연결하게 한다
- D. Aurora Global Database를 구성한다

**Q37.** 누군가 보안 그룹에 `0.0.0.0/0`으로 22번 포트를 열면 **자동으로 그 규칙을 제거**하고 기록을 남기고 싶다. 운영 부담이 가장 적은 방법은?

- A. GuardDuty 결과를 보고 사람이 지운다
- B. IAM 정책으로 `ec2:AuthorizeSecurityGroupIngress`를 모두 거부한다
- C. AWS Config 관리형 규칙(`restricted-ssh`)과 Systems Manager Automation 문서를 이용한 **자동 교정(remediation)** 을 설정한다
- D. 매일 밤 모든 보안 그룹을 검사하는 스크립트를 EC2 cron으로 돌린다

**Q38.** IoT 이벤트(JSON)를 **수 분 안에** S3에 Parquet로 적재해 Athena로 조회하려 한다. 재처리·다중 소비자는 필요 없다. **운영 부담이 가장 적은** 방법은?

- A. Amazon Data Firehose에 레코드 형식 변환(Glue 데이터 카탈로그 스키마 사용)을 켜고 S3를 대상으로 한다
- B. Kinesis Data Streams + EC2의 KCL 애플리케이션이 Parquet로 변환해 S3에 쓴다
- C. SQS + Glue ETL 작업을 5분마다 실행한다
- D. Kinesis Data Streams + Lambda가 Parquet 라이브러리로 변환해 S3에 쓴다

**Q39.** S3의 CSV 로그(하루 2TB)를 Athena로 조회하는데, 쿼리마다 전체 데이터를 스캔해 느리고 비싸다. 대부분의 쿼리는 날짜 조건과 열 몇 개만 쓴다. 가장 효과적인 개선 **2가지**는? (2개 선택)

- A. CSV 파일을 하나의 큰 파일로 합친다
- B. 날짜 기준으로 **파티션**(예: `dt=2026-11-19/`)을 나누고 쿼리에 파티션 조건을 쓴다
- C. Athena 쿼리 결과 버킷을 S3 Standard-IA로 바꾼다
- D. 데이터를 **Parquet** 같은 열 기반·압축 형식으로 변환한다
- E. S3 Transfer Acceleration을 켠다

**Q40.** 온프레미스에서 **8GB** 백업 파일을 S3로 올리는 스크립트가 단일 `PutObject` 호출로 실패한다. 회선은 충분하다. 올바른 방법은?

- A. 버킷을 다른 리전으로 옮긴다
- B. Snowball Edge를 주문한다
- C. S3 Transfer Acceleration을 켠다
- D. 멀티파트 업로드로 나눠 올린다(AWS CLI `aws s3 cp`는 자동으로 멀티파트 사용)

**Q41.** 온프레미스 데이터센터의 서버가 매일 S3에 백업을 올린다. 지금은 IAM 사용자 액세스 키를 서버에 저장해 쓴다. 회사에는 사내 사설 CA로 발급한 서버 인증서가 이미 있다. 장기 키를 없애려면?

- A. 액세스 키를 Secrets Manager에 넣고 서버가 읽게 한다
- B. IAM Roles Anywhere에 사설 CA를 신뢰 앵커로 등록하고, 서버가 인증서로 임시 자격 증명을 받아 역할을 맡게 한다
- C. S3 버킷을 회사 공인 IP에만 퍼블릭으로 연다
- D. 서버를 EC2로 옮길 때까지 키를 30일마다 교체한다

**Q42.** 소프트웨어 회사는 S3에서 업데이트 파일(500MB)을 전 세계 고객 장비에 직접 내려준다. 월 **데이터 송신(DTO) 요금**이 크고, 같은 파일이 수십만 번 다운로드된다. **가장 비용 효율적인** 개선은?

- A. S3 Transfer Acceleration을 켠다
- B. 모든 리전에 버킷을 복제한다
- C. S3 앞에 CloudFront를 두고(OAC), 업데이트 파일에 긴 캐시 TTL을 준다
- D. 업데이트 파일을 S3 One Zone-IA로 옮긴다

**Q43.** 계정 A의 S3 버킷 객체는 계정 A의 **고객 관리형 KMS 키**로 SSE-KMS 암호화되어 있다. 계정 B의 애플리케이션 역할이 이 객체를 읽으려 하면 `AccessDenied`가 난다. 계정 B 역할에는 이미 `s3:GetObject`와 `kms:Decrypt`를 허용하는 IAM 정책이 있다. 계정 A에서 해야 할 일을 **2개** 고르시오.

- A. 계정 B에 같은 이름의 KMS 키를 만든다
- B. 버킷 정책에서 계정 B 역할에 `s3:GetObject`를 허용한다
- C. 버킷에 Requester Pays를 켠다
- D. KMS 키 정책에서 계정 B(또는 그 역할)에 `kms:Decrypt`를 허용한다
- E. 객체를 AWS 관리형 키(`aws/s3`)로 다시 암호화한다

**Q44.** 규정상 분기마다 찍은 EBS 스냅숏을 **7년 보관**해야 한다. 복원은 감사 때만 하며 **며칠 걸려도 괜찮다**. 스냅숏 저장 비용을 가장 줄이는 방법은?

- A. 스냅숏을 다른 리전으로 복사한다
- B. 스냅숏에서 볼륨을 만들어 sc1으로 보관한다
- C. 분기 스냅숏을 **EBS Snapshots Archive** 계층으로 옮긴다
- D. Data Lifecycle Manager로 매일 스냅숏을 찍는다

**Q45.** 운영 서비스의 공유 파일(EFS)을 비용 절감 목적으로 **EFS One Zone**으로 옮기자는 제안이 나왔다. 요구 사항은 "AZ 하나가 장애 나도 파일에 계속 접근"이다. 올바른 판단은?

- A. EBS Multi-Attach로 바꾼다
- B. 요구 사항을 어기므로 EFS 리전(Standard) 스토리지를 유지하고, 비용은 수명 주기 관리(IA·Archive)로 줄인다
- C. One Zone으로 옮기고 AWS Backup으로 매일 백업하면 요구를 만족한다
- D. One Zone으로 옮기고 모든 AZ의 인스턴스가 마운트하면 요구를 만족한다

**Q46.** SQS 표준 큐를 소비하는 워커의 처리 시간은 보통 20초지만 가끔 2분까지 걸린다. 큐의 가시성 제한 시간은 기본값(30초)이다. 같은 주문이 **두 번 처리되는** 사고가 가끔 난다. 가장 알맞은 조치는? (워커는 이미 멱등성 키를 쓰고 있지 않다)

- A. 가시성 제한 시간을 최대 처리 시간보다 길게(예: 3분) 늘리고, 오래 걸리는 메시지는 `ChangeMessageVisibility`로 연장한다
- B. 큐를 FIFO로 바꾸기만 하면 중복이 사라진다
- C. 메시지 보존 기간을 14일로 늘린다
- D. 긴 폴링 대기 시간을 20초로 늘린다

**Q47.** 주문 내역 DynamoDB 테이블(20TB)은 대부분 1년 넘은 항목이고 거의 읽지 않는다. 비용의 대부분이 **저장 비용**이다. 애플리케이션 변경 없이 비용을 줄이는 가장 간단한 방법은?

- A. DAX를 추가한다
- B. 온디맨드 모드로 바꾼다
- C. 글로벌 보조 인덱스를 모두 지운다
- D. 테이블 클래스를 **DynamoDB Standard-IA**로 바꾼다

**Q48.** EKS 클러스터의 노드 IAM 역할에 S3·DynamoDB·SQS 권한을 모두 넣어 두었더니, 모든 Pod가 모든 권한을 갖게 되었다. **Pod(서비스 계정)마다** 필요한 권한만 주려면?

- A. EKS Pod Identity(또는 IRSA)로 Kubernetes 서비스 계정마다 IAM 역할을 연결한다
- B. 노드 역할에서 권한을 빼고 보안 그룹으로 제어한다
- C. 노드 그룹을 서비스마다 따로 만들고 노드 역할을 나눈다
- D. Pod에 IAM 사용자 액세스 키를 Kubernetes Secret으로 넣는다

**Q49.** 영상 편집 팀의 Windows 워크스테이션(EC2)들이 **SMB**로 대용량 프로젝트 파일을 공유하며, 회사 Active Directory로 권한을 관리한다. 지연이 짧고 처리량이 높아야 한다. 알맞은 서비스는?

- A. Amazon S3 + Storage Gateway S3 File Gateway
- B. Amazon FSx for Windows File Server(SSD 스토리지)
- C. Amazon FSx for Lustre
- D. Amazon EFS 최대 처리량 모드

**Q50.** 감사인은 특정 S3 버킷에서 **어떤 IAM 주체가 어떤 객체를 언제 읽고(GetObject) 지웠는지** 기록을 요구한다. 계정에는 기본 설정의 CloudTrail 추적이 있다. 무엇을 해야 하는가?

- A. AWS Config로 버킷 설정 이력을 기록한다
- B. VPC 흐름 로그를 켠다
- C. 추적을 다중 리전으로 바꾸면 자동으로 기록된다
- D. CloudTrail 추적에 해당 버킷의 **S3 데이터 이벤트** 기록을 추가한다

**Q51.** RDS for PostgreSQL을 쓰는 회사의 DR 요구는 **RPO 5분 이내, RTO 30분 이내**이며 **가장 비용 효율적**이어야 한다. 가장 알맞은 방법은?

- A. Multi-AZ DB 클러스터로 바꾼다
- B. 다른 리전에 **리전 간 읽기 전용 복제본**을 두고, 재해 시 승격해 애플리케이션 엔드포인트를 바꾼다
- C. 매일 자동 스냅숏을 다른 리전으로 복사한다
- D. Aurora PostgreSQL로 옮기고 Aurora Global Database를 구성한다

**Q52.** 배달 앱의 주문 상태 변경 이벤트는 **주문별로 순서대로** 처리되어야 하고, 전체 처리량은 초당 수천 건이다. 개발자가 SQS FIFO 큐를 만들고 모든 메시지에 `MessageGroupId=orders`를 넣었더니 처리가 심하게 밀린다. 가장 알맞은 수정은?

- A. SQS 표준 큐로 바꾸고 소비자를 늘린다
- B. 가시성 제한 시간을 줄인다
- C. `MessageGroupId`를 **주문 ID**로 바꾸고, 높은 처리량 모드를 켠다
- D. 메시지를 10개씩 배치로 보내기만 한다

**Q53.** Organizations의 모든 계정에서 S3 버킷이 **실수로라도** 퍼블릭이 되지 않게 하려 한다. 새 계정이 생겨도 자동 적용되어야 하고 운영 부담은 최소화해야 한다. 가장 알맞은 방법은?

- A. 각 버킷의 버킷 정책에 퍼블릭 거부 문장을 넣는다
- B. 각 계정에 계정 수준 S3 퍼블릭 액세스 차단을 켜고, `s3:PutAccountPublicAccessBlock` 변경을 거부하는 SCP를 루트에 연결한다(새 계정은 Control Tower·자동화로 기본 설정)
- C. AWS Config 규칙으로 퍼블릭 버킷을 탐지해 이메일을 보낸다
- D. Macie로 퍼블릭 버킷을 찾는다

**Q54.** RDS for MySQL 마스터 비밀번호를 정기적으로 교체해야 한다. 팀은 교체 로직을 직접 작성·유지하고 싶지 않다. **운영 부담이 가장 적은** 방법은?

- A. Parameter Store SecureString에 저장하고 EventBridge Scheduler + Lambda로 교체한다
- B. Secrets Manager에 비밀을 만들고 교체용 Lambda를 직접 작성한다
- C. RDS 인스턴스에서 마스터 사용자 비밀번호를 Secrets Manager로 관리하도록 설정한다(RDS 통합 관리형 교체)
- D. IAM 데이터베이스 인증으로 바꾸고 마스터 비밀번호를 그대로 둔다

**Q55.** 사내 관리 도구(ALB 뒤 EC2)에 회사 IdP(OIDC 지원)로 **로그인한 직원만** 접근하게 하려 한다. 애플리케이션 코드를 거의 고치지 않으려면?

- A. AWS WAF로 쿠키 이름을 검사한다
- B. ALB 리스너 규칙에 `authenticate-oidc` 작업을 추가해 IdP와 연동한다
- C. 애플리케이션에 OIDC 라이브러리를 넣고 로그인 화면을 만든다
- D. 보안 그룹에서 회사 공인 IP만 허용한다

**Q56.** 증권사의 핵심 거래 시스템은 온프레미스와 AWS 사이 연결이 **어떤 단일 장애(장비·회선·DX 위치)** 에도 끊기면 안 되고, 장애 시에도 **같은 대역폭**이 유지되어야 한다. 알맞은 구성은?

- A. Site-to-Site VPN 터널 두 개만 쓴다
- B. 서로 다른 **두 DX 위치**에 각각 독립된 장비로 연결을 두 개 이상 만든다(최대 복원력 모델)
- C. 한 DX 위치에 연결 두 개를 만든다
- D. DX 연결 하나와 Site-to-Site VPN 백업을 둔다

**Q57.** 상태 없는(stateless) 웹 계층을 Auto Scaling 그룹으로 운영한다. 비용을 줄이려고 일부를 Spot으로 돌리되, **Spot 회수 때도 서비스 용량이 크게 줄지 않게** 하려 한다. 가장 알맞은 구성은?

- A. Spot 최고 가격을 온디맨드의 10배로 설정한다
- B. 혼합 인스턴스 정책으로 온디맨드 기준 용량을 두고 나머지를 Spot으로, **여러 인스턴스 유형·AZ**와 price-capacity-optimized 할당, 용량 재조정(Capacity Rebalancing)을 켠다
- C. 모든 인스턴스를 같은 유형의 Spot으로 바꾼다
- D. Spot 인스턴스를 Dedicated Hosts에 띄운다

**Q58.** Kinesis Data Streams 소비자 애플리케이션에 버그가 생겨 이틀 동안 데이터를 잘못 처리했다. 버그를 고친 뒤 **이틀 치를 다시 처리**하려 했지만 데이터가 이미 사라졌다. 앞으로 같은 일이 생겨도 최대 7일 치를 다시 처리하려면?

- A. 스트림의 **데이터 보존 기간을 7일 이상**으로 늘린다
- B. 소비자를 Lambda로 바꾼다
- C. 향상된 팬아웃(enhanced fan-out) 소비자로 바꾼다
- D. 샤드 수를 두 배로 늘린다

**Q59.** Compute Optimizer가 메모리 사용량이 높은 서버에도 "다운사이즈 가능"이라고 추천해, 팀이 추천을 믿지 못한다. 추천 정확도를 높이는 **가장 직접적인** 방법은?

- A. Trusted Advisor 비용 최적화 검사를 새로 고친다
- B. Compute Optimizer 향상된 인프라 지표(유료)만 켠다
- C. 인스턴스에 세부 모니터링(1분)을 켠다
- D. 인스턴스에 CloudWatch 에이전트를 설치해 **메모리 사용률** 지표를 보내게 한다

**Q60.** 글로벌 메모 앱의 사용자는 서울·프랑크푸르트·버지니아에 고르게 있고, 모두 자기 데이터에 **쓰기 지연이 짧아야** 한다. 데이터 모델은 사용자 ID 기준 키-값이고, 같은 항목을 여러 리전에서 동시에 수정하는 일은 거의 없다. 가장 알맞은 방법은?

- A. DynamoDB 글로벌 테이블로 세 리전에 복제본을 두고 각 리전 애플리케이션이 가까운 복제본에 쓴다
- B. Aurora Global Database로 세 리전을 구성한다
- C. 버지니아 테이블 하나 + CloudFront로 API 캐시
- D. 버지니아 DynamoDB 테이블 하나 + 각 리전에 DAX

**Q61.** Aurora MySQL 클러스터의 청구서를 보니 **I/O 요청 요금이 전체 Aurora 비용의 절반**을 넘는다. 워크로드는 쓰기·읽기가 많은 OLTP이며 바뀔 계획이 없다. **가장 비용 효율적인** 조치는?

- A. 스토리지를 gp3로 바꾼다
- B. 읽기 전용 복제본을 추가한다
- C. Aurora Serverless v2로 바꾼다
- D. 클러스터 스토리지 구성을 **Aurora I/O-Optimized**로 바꾼다

**Q62.** 샌드박스 계정은 월 300달러 예산을 넘으면 **사람 개입 없이** 새 리소스 생성을 막아야 한다(기존 리소스는 유지). 운영 부담이 가장 적은 방법은?

- A. AWS Budgets 예산 작업으로 실제 비용이 예산을 넘으면 생성 작업을 거부하는 **SCP(또는 IAM 정책)를 자동 적용**한다
- B. CloudWatch 결제 경보로 관리자에게 SMS를 보낸다
- C. Cost Anomaly Detection 알림을 Lambda로 받아 계정을 닫는다
- D. Cost Explorer 보고서를 매일 아침 확인한다

**Q63.** Python으로 작성된 Lambda 함수(순수 Python 의존성만 사용)가 월 수억 번 실행된다. **코드 변경 없이** 컴퓨팅 비용을 줄이는 가장 간단한 방법은?

- A. 함수를 VPC에 연결한다
- B. 메모리를 최솟값(128MB)으로 줄인다
- C. 함수 아키텍처를 **arm64(Graviton)** 로 바꾸고 테스트한다
- D. 프로비저닝된 동시성을 켠다

**Q64.** 보안팀은 조직 안의 S3 버킷·KMS 키·IAM 역할·Lambda 함수 중 **조직 밖의 주체에게 접근이 열린 것**을 자동으로 찾고 싶다. 알맞은 서비스는?

- A. Amazon Inspector
- B. IAM Access Analyzer(조직을 신뢰 영역으로 하는 외부 접근 분석기)
- C. Amazon GuardDuty
- D. AWS Trusted Advisor

**Q65.** IoT 플랫폼은 장치 상태를 **4KB짜리 S3 객체**로 매일 수천만 개씩 쌓는다. 30일이 지나면 거의 읽지 않고 3년 보관한다. 비용을 줄이려고 "30일 후 Standard-IA로 전환" 규칙을 제안받았다. **가장 비용 효율적인** 방법은?

- A. S3 Intelligent-Tiering에 저장한다
- B. 30일 후 S3 Glacier Instant Retrieval로 전환한다
- C. 제안대로 30일 후 Standard-IA로 전환한다
- D. 작은 객체를 시간 단위 등으로 **묶어 큰 객체(예: 압축 아카이브·Parquet)** 로 만든 뒤 저렴한 클래스로 전환한다


> ⏱️ **여기까지가 시험입니다.** 남은 시간에 `?` 표시 문항을 검토하세요.

---

## 4. 정답표

| 문항 | 정답 | 도메인 | 주제 | 문항 | 정답 | 도메인 | 주제 |
|---|---|---|---|---|---|---|---|
| Q1 | **C** | 복원력 | 기존 객체 복제 | Q34 | **C** | 고성능 | 고정 IP + 빠른 리전 전환 API |
| Q2 | **B,E** | 복원력 | 하이브리드 DNS (2개 선택) | Q35 | **A,E** | 복원력 | Route 53 액티브-패시브 (2개 선택) |
| Q3 | **A** | 고성능 | TLS 패스스루 | Q36 | **C** | 복원력 | 장애 조치 중 연결 오류 |
| Q4 | **A** | 비용 최적화 | 개발 환경 NAT | Q37 | **C** | 보안 | SSH 전체 개방 자동 교정 |
| Q5 | **A** | 비용 최적화 | VPC 3개 연결 비용 | Q38 | **A** | 고성능 | S3로 Parquet 적재 — 운영 부담 |
| Q6 | **D** | 비용 최적화 | RDS 장기 약정 — 최대 할인 | Q39 | **B,D** | 고성능 | Athena 쿼리 비용·속도 |
| Q7 | **C** | 보안 | EBS 기본 암호화 | Q40 | **D** | 고성능 | 대용량 업로드 한계 |
| Q8 | **A** | 고성능 | Java Lambda 콜드 스타트 | Q41 | **B** | 보안 | 온프레미스 서버의 임시 자격 증명 |
| Q9 | **A** | 보안 | TLS 강제 | Q42 | **C** | 비용 최적화 | 다운로드 송신 비용 |
| Q10 | **D** | 보안 | 서드파티 역할 위임 | Q43 | **B,D** | 보안 | 교차 계정 SSE-KMS 객체 읽기 (2개 선택) |
| Q11 | **B** | 복원력 | 긴 사람 승인 워크플로 | Q44 | **C** | 비용 최적화 | 스냅숏 장기 보관 |
| Q12 | **C** | 복원력 | 세션 캐시 고가용성 | Q45 | **B** | 복원력 | EFS 비용과 복원력 |
| Q13 | **B,D** | 복원력 | 랜섬웨어 대비 백업 (2개 선택) | Q46 | **A** | 복원력 | SQS 중복 처리 |
| Q14 | **A** | 보안 | 도메인 기반 아웃바운드 필터링 | Q47 | **D** | 비용 최적화 | 저장 비용이 큰 DynamoDB 테이블 |
| Q15 | **D** | 보안 | CloudTrail 끄기 방지 | Q48 | **A** | 보안 | EKS Pod별 권한 |
| Q16 | **A** | 보안 | VPC 안에서만 부르는 API | Q49 | **B** | 고성능 | Windows 공유 스토리지 성능 |
| Q17 | **C** | 복원력 | 소비자별 필터링 팬아웃 | Q50 | **D** | 보안 | 객체 수준 감사 |
| Q18 | **D** | 고성능 | S3 요청 속도 | Q51 | **B** | 복원력 | 리전 간 DR — 비용과 RPO |
| Q19 | **C** | 고성능 | DynamoDB 핫 파티션 | Q52 | **C** | 복원력 | 고객별 순서 + 처리량 |
| Q20 | **A** | 고성능 | Aurora 읽기 자동 확장 | Q53 | **B** | 보안 | 조직 전체 S3 퍼블릭 차단 |
| Q21 | **D** | 고성능 | 실시간 순위표 | Q54 | **C** | 보안 | RDS 비밀번호 교체 — 운영 부담 |
| Q22 | **C** | 복원력 | ASG 상태 확인 유형 | Q55 | **B** | 보안 | ALB 로그인 — 코드 최소 |
| Q23 | **A** | 보안 | 보안 결과 중앙 집계 | Q56 | **B** | 복원력 | Direct Connect 최대 복원력 |
| Q24 | **A** | 복원력 | Lambda 비동기 실패 정보 | Q57 | **B** | 비용 최적화 | Spot 웹 계층 |
| Q25 | **C** | 고성능 | CloudFront 캐시 적중률 | Q58 | **A** | 복원력 | Kinesis 재처리 여유 |
| Q26 | **B,E** | 보안 | VPC Lambda의 S3·DynamoDB 접근 (2개 선택) | Q59 | **D** | 비용 최적화 | 메모리 기반 rightsizing |
| Q27 | **B,E** | 비용 최적화 | 섞인 워크로드 구매 전략 (2개 선택) | Q60 | **A** | 고성능 | 글로벌 저지연 쓰기 |
| Q28 | **A** | 보안 | 회원 전용 대용량 다운로드 | Q61 | **D** | 비용 최적화 | Aurora I/O 요금 |
| Q29 | **C** | 고성능 | EBS IOPS — 가장 싼 방법 | Q62 | **A** | 비용 최적화 | 예산 초과 시 자동 차단 |
| Q30 | **B** | 보안 | 설정값 저장 — 가장 싼 방법 | Q63 | **C** | 비용 최적화 | Lambda 비용 — 코드 변경 없이 |
| Q31 | **D** | 보안 | 계층 간 보안 그룹 | Q64 | **B** | 보안 | 의도치 않은 외부 공유 찾기 |
| Q32 | **B** | 복원력 | DR 전략 고르기 | Q65 | **D** | 비용 최적화 | 작은 객체 수억 개 |
| Q33 | **D** | 고성능 | 같은 AZ 공유 블록 볼륨 |  |  |  |  |

---

## 5. 해설

> 함정 난이도 회차라 해설마다 **"왜 그럴듯한 오답이 틀렸나"** 를 함께 적었습니다. 📎 링크는 근거가 되는 AWS 공식 문서입니다.

<details>
<summary><b>Q1 ~ Q10 해설</b></summary>

**Q1 — 정답 C** · 복원력 · 기존 객체 복제

복제 규칙은 **설정 이후** 새로 쓰인 객체에 적용된다. 기존 객체·이전에 실패한 객체는 **S3 Batch Replication**으로 복제한다. DataSync도 복사는 되지만 복제 메타데이터·상태 관리가 달라 S3 네이티브 방식이 운영 부담이 적다.

📎 [S3 Batch Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-batch-replication-batch.html)

**Q2 — 정답 B,E** · 복원력 · 하이브리드 DNS (2개 선택)

Resolver 인바운드 엔드포인트는 온프레미스에서 VPC DNS로 들어오는 질의를, 아웃바운드 엔드포인트와 전달 규칙은 VPC에서 온프레미스 DNS로 나가는 질의를 처리한다. DHCP 옵션만 바꾸면 VPC 안에서 프라이빗 호스팅 영역 조회가 깨진다.

📎 [Route 53 Resolver](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)

**Q3 — 정답 A** · 고성능 · TLS 패스스루

ALB HTTPS 리스너와 NLB **TLS 리스너**는 모두 로드 밸런서에서 TLS를 **종료(복호화)** 한다. 복호화 금지 조건에서는 NLB **TCP 리스너**로 패스스루해야 한다. GWLB는 방화벽 어플라이언스 삽입용이다.

📎 [NLB 리스너](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-listeners.html)

**Q4 — 정답 A** · 비용 최적화 · 개발 환경 NAT

트래픽이 적으면 AZ 간 전송 요금보다 **NAT 시간 요금**이 크므로 하나로 줄이는 것이 싸고, AZ 장애 중단도 허용된다. 운영 환경이라면 답이 반대가 된다(W04 문제 10). 퍼블릭 IP를 붙이면 보안이 나빠지고 퍼블릭 IPv4도 시간당 과금된다. TGW는 연결·처리 요금이 추가된다.

📎 [NAT 게이트웨이 요금](https://aws.amazon.com/vpc/pricing/)

**Q5 — 정답 A** · 비용 최적화 · VPC 3개 연결 비용

VPC 피어링은 **GB당 처리 요금이 없다**(AZ 간 전송 요금만 적용). Transit Gateway는 운영이 단순하고 확장에 유리하지만 **처리 GB당 요금**이 붙어 대량 전송에서 비싸다. VPC가 3개뿐이고 늘릴 계획이 없다는 조건이 피어링을 정답으로 만든다.

📎 [VPC 요금(피어링 데이터 전송)](https://aws.amazon.com/vpc/pricing/) · [Transit Gateway 요금](https://aws.amazon.com/transit-gateway/pricing/)

**Q6 — 정답 D** · 비용 최적화 · RDS 장기 약정 — 최대 할인

Compute Savings Plans는 **RDS에 적용되지 않는다**(EC2·Fargate·Lambda). 2025-12에 나온 Database Savings Plans는 RDS에도 쓰이지만 **1년 약정**이고 프로비저닝 인스턴스 할인 폭이 RI보다 작다(AWS 발표 기준 최대 20%). 구성이 3년 고정이면 3년 RI가 가장 크게 할인된다. 운영 DB를 밤에 중지하는 것은 요구에 맞지 않는다.

📎 [RDS 예약 인스턴스](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithReservedDBInstances.html) · [Database Savings Plans 발표](https://aws.amazon.com/about-aws/whats-new/2025/12/database-savings-plans-savings)

**Q7 — 정답 C** · 보안 · EBS 기본 암호화

EBS 기본 암호화를 켜면 그 리전에서 새로 만드는 볼륨이 **자동으로 암호화**된다(리전 단위 설정이라는 점이 함정 포인트). Config는 탐지일 뿐이고, 생성 자체를 막으면 업무가 멈춘다. 기존 볼륨은 자동으로 암호화되지 않는다.

📎 [EBS 기본 암호화](https://docs.aws.amazon.com/ebs/latest/userguide/encryption-by-default.html)

**Q8 — 정답 A** · 고성능 · Java Lambda 콜드 스타트

SnapStart는 초기화가 끝난 실행 환경의 스냅숏을 캐시해 콜드 스타트를 줄이며, **Java 런타임에는 추가 요금이 없다**. 프로비저닝된 동시성도 효과적이지만 켜 둔 시간만큼 요금이 붙는다(조건 위반). 메모리 증가는 초기화를 다소 빠르게 하지만 비용이 늘고 근본 해결이 아니다.

📎 [Lambda SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html)

**Q9 — 정답 A** · 보안 · TLS 강제

`aws:SecureTransport` 조건은 요청이 TLS로 왔는지를 나타낸다. 기본 암호화(SSE)는 **저장 시** 암호화이므로 전송 구간과 무관하다. 퍼블릭 액세스 차단·ACL은 접근 주체에 관한 설정이다.

📎 [S3 보안 모범 사례](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)

**Q10 — 정답 D** · 보안 · 서드파티 역할 위임

외부 ID는 업체가 **고객마다 고유하게** 관리하는 값으로, AssumeRole 요청에 포함되어야만 역할을 맡을 수 있게 해 혼동된 대리자 문제를 막는다. 나머지는 위험을 줄이지 못하거나(이름 숨기기) 다른 문제를 다룬다.

📎 [외부 ID와 혼동된 대리자](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html)

</details>

<details>
<summary><b>Q11 ~ Q20 해설</b></summary>

**Q11 — 정답 B** · 복원력 · 긴 사람 승인 워크플로

Express 워크플로는 **최대 5분**까지 실행되므로 5일 대기가 불가능하다(이름만 보고 고르기 쉬운 함정). Standard는 최대 1년 실행되고 실행 이력을 보관한다. SQS 지연 큐는 최대 15분, Lambda는 최대 15분 실행이다.

📎 [Standard vs Express 워크플로](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html)

**Q12 — 정답 C** · 복원력 · 세션 캐시 고가용성

Memcached는 노드를 여러 개 둬도 **데이터를 복제하지 않아** 노드가 사라지면 그 노드의 키를 잃는다(노드가 여러 AZ에 있다는 말에 속기 쉽다). Valkey·Redis OSS는 복제본과 Multi-AZ 자동 장애 조치를 지원한다.

📎 [ElastiCache Multi-AZ](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/AutoFailover.html)

**Q13 — 정답 B,D** · 복원력 · 랜섬웨어 대비 백업 (2개 선택)

Vault Lock 규정 준수 모드는 잠금 이후 **루트 사용자도** 보존 정책을 바꾸거나 복구 지점을 지울 수 없다. 다른 계정으로 복사하면 원본 계정 탈취와 격리된다. 같은 계정의 IAM 정책은 탈취된 관리자가 바꿀 수 있다.

📎 [AWS Backup Vault Lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)

**Q14 — 정답 A** · 보안 · 도메인 기반 아웃바운드 필터링

보안 그룹·NACL은 **IP·포트** 기반이라 도메인 이름을 다루지 못한다. WAF는 CloudFront·ALB·API Gateway 같은 **인바운드 HTTP** 리소스에 붙으며 NAT 게이트웨이에는 붙지 않는다. Network Firewall은 도메인 목록 규칙으로 아웃바운드를 제어하고, 중앙 검사 VPC 패턴을 지원한다.

📎 [Network Firewall 도메인 필터링](https://docs.aws.amazon.com/network-firewall/latest/developerguide/stateful-rule-groups-domain-names.html)

**Q15 — 정답 D** · 보안 · CloudTrail 끄기 방지

계정 관리자는 자기 IAM 정책을 바꿀 수 있으므로 IAM 정책은 "관리자 포함"을 막지 못한다. **SCP는 멤버 계정의 최대 권한**이라 관리자도 넘을 수 없다. 조직 추적은 멤버 계정에서 수정·삭제할 수 없다는 장점도 있다. Config 규칙은 사후 **탐지**이고, Object Lock은 로그 파일 삭제를 막을 뿐 기록 중지를 막지 못한다.

📎 [조직 추적](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html) · [SCP](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)

**Q16 — 정답 A** · 보안 · VPC 안에서만 부르는 API

프라이빗 REST API는 **인터페이스 VPC 엔드포인트를 통해서만** 접근할 수 있고, 리소스 정책으로 특정 VPC·엔드포인트만 허용한다. 리전·엣지 API에 WAF·API 키를 붙이면 막을 수는 있어도 엔드포인트가 인터넷에 노출된다. VPC 클라이언트는 프라이빗 IP로 오므로 WAF에서 VPC CIDR을 판별하는 것 자체도 맞지 않는다.

📎 [API Gateway 프라이빗 API](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html)

**Q17 — 정답 C** · 복원력 · 소비자별 필터링 팬아웃

SNS 구독 필터 정책은 메시지 속성(또는 본문)을 보고 **구독별로** 전달 여부를 정한다. 토픽 분리도 동작하지만 발행자 코드가 바뀐다. 소비자 쪽에서 버리는 것은 낭비와 코드 추가다.

📎 [SNS 메시지 필터링](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)

**Q18 — 정답 D** · 고성능 · S3 요청 속도

S3는 **접두사당** 초당 PUT 3,500건·GET 5,500건 이상을 처리하며, 접두사를 늘리면 처리량이 늘어난다. Transfer Acceleration은 거리 문제를 푸는 기능이다.

📎 [S3 성능 최적화](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html)

**Q19 — 정답 C** · 고성능 · DynamoDB 핫 파티션

테이블 용량이 남는데 스로틀링이 나면 **특정 파티션에 쓰기가 몰린** 것이다. 파티션 키를 분산(쓰기 샤딩)해야 한다. 온디맨드·적응형 용량도 도움이 되지만 단일 파티션 처리 한도를 넘는 집중은 해결하지 못한다. DAX는 읽기 캐시다.

📎 [파티션 키 설계](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-sharding.html)

**Q20 — 정답 A** · 고성능 · Aurora 읽기 자동 확장

Aurora Auto Scaling은 복제본 수를 정책에 따라 조절하고, 리더 엔드포인트는 사용 가능한 복제본으로 연결을 분산한다. 복제본을 고정 15개 두면 유휴 시간에 비용이 낭비되고 인스턴스 엔드포인트를 애플리케이션이 관리해야 한다.

📎 [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html)

</details>

<details>
<summary><b>Q21 ~ Q30 해설</b></summary>

**Q21 — 정답 D** · 고성능 · 실시간 순위표

정렬된 집합은 점수 갱신과 순위 조회를 로그 시간에 처리하는 **순위표의 대표 자료 구조**다. 복제본을 늘려도 매번 정렬 쿼리를 돌리는 구조는 그대로다. DynamoDB Scan은 비싸고 느리다.

📎 [ElastiCache 사용 사례](https://aws.amazon.com/elasticache/redis/)

**Q22 — 정답 C** · 복원력 · ASG 상태 확인 유형

Auto Scaling 그룹의 기본 상태 확인은 **EC2 상태 검사**(하드웨어·OS 수준)라 애플리케이션 오류를 모른다. ELB 상태 확인을 켜면 대상 그룹에서 비정상인 인스턴스를 교체한다.

📎 [Auto Scaling 상태 확인](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-health-checks.html)

**Q23 — 정답 A** · 보안 · 보안 결과 중앙 집계

Security Hub는 여러 보안 서비스의 결과를 **표준 형식으로 집계**하고, 위임 관리자로 조직 전체를 한곳에서 보며, 보안 표준(CSPM) 점검을 제공한다. Detective는 조사 도구다.

📎 [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)

**Q24 — 정답 A** · 복원력 · Lambda 비동기 실패 정보

Lambda **대상(destinations)** 은 호출 결과 레코드(요청 페이로드, 응답·오류 정보 등)를 보내 DLQ보다 많은 맥락을 준다. DLQ도 실패 이벤트를 보존하지만 원래 이벤트 위주라 "실패 맥락" 요구에는 대상이 더 맞다. AWS 문서도 새 구성에는 대상을 권장한다.

📎 [Lambda 비동기 호출 대상](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async-retain-records.html)

**Q25 — 정답 C** · 고성능 · CloudFront 캐시 적중률

캐시 키에 불필요한 값이 들어가면 같은 콘텐츠가 서로 다른 객체로 캐시되어 적중률이 떨어진다. **캐시 정책**으로 필요한 값만 키에 넣고, 오리진에 전달만 하면 되는 값은 **오리진 요청 정책**으로 분리한다. 오리진 확장은 증상 완화일 뿐이다.

📎 [캐시 키 제어](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/controlling-the-cache-key.html)

**Q26 — 정답 B,E** · 보안 · VPC Lambda의 S3·DynamoDB 접근 (2개 선택)

S3와 DynamoDB는 **무료 게이트웨이 엔드포인트**를 지원한다. 인터페이스 엔드포인트도 동작하지만 AZ당 시간 요금과 GB 요금이 붙어 "추가 비용이 가장 적게"를 어긴다. Lambda를 VPC에서 빼면 S3·DynamoDB에는 닿지만, 함수가 VPC에 붙은 데는 이유(다른 프라이빗 리소스 접근)가 있다고 보는 것이 출제 의도다. NAT는 조건 위반이다.

📎 [게이트웨이 엔드포인트](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html)

**Q27 — 정답 B,E** · 비용 최적화 · 섞인 워크로드 구매 전략 (2개 선택)

약정은 **기준(baseline) 사용량**만큼만 하고, 그 위의 급증은 온디맨드로 처리하는 것이 원칙이다. 최대 사용량만큼 약정하면 평소에 쓰지 않는 약정분을 낸다(그럴듯한 함정). 재시도 가능한 배치는 Spot이 가장 싸다.

📎 [Savings Plans](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html)

**Q28 — 정답 A** · 보안 · 회원 전용 대용량 다운로드

S3 presigned URL도 "회원만·기간 한정"은 만족하지만 **캐시가 없어** 같은 파일을 매번 S3에서 내보내 지연·비용 요구를 놓친다(그럴듯한 오답). CloudFront 서명된 URL은 엣지 캐시와 접근 제어를 함께 준다. Global Accelerator는 S3를 엔드포인트로 받지 않고 캐시도 없다.

📎 [CloudFront 서명된 URL](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html)

**Q29 — 정답 C** · 고성능 · EBS IOPS — 가장 싼 방법

gp2는 GiB당 3 IOPS라 IOPS를 얻으려면 **용량을 키워야** 한다. gp3는 크기와 무관하게 IOPS·처리량을 따로 지정할 수 있어 같은 성능을 훨씬 싸게 낸다. io2도 동작하지만 이 수준에서는 gp3보다 비싸다(성능이 넘치는 오답).

📎 [EBS 볼륨 유형](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html)

**Q30 — 정답 B** · 보안 · 설정값 저장 — 가장 싼 방법

교체가 필요 없고 비용이 중요하면 Parameter Store **표준 파라미터(추가 요금 없음)** + SecureString이 맞다. Secrets Manager는 자동 교체가 강점이지만 **비밀당 월 요금**이 붙는다. 평문 환경 변수는 "암호화" 요구를 어긴다(Lambda 환경 변수는 저장 시 암호화되긴 하지만 설정 콘솔·API에서 평문으로 보인다).

📎 [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) · [Secrets Manager 요금](https://aws.amazon.com/secrets-manager/pricing/)

</details>

<details>
<summary><b>Q31 ~ Q40 해설</b></summary>

**Q31 — 정답 D** · 보안 · 계층 간 보안 그룹

보안 그룹은 **다른 보안 그룹을 출발지로 참조**할 수 있어, ALB 노드 IP가 바뀌어도 규칙을 고칠 필요가 없다. VPC CIDR 전체 허용은 범위가 너무 넓고, IP 나열은 ALB IP가 바뀌면 깨진다.

📎 [보안 그룹 규칙](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)

**Q32 — 정답 B** · 복원력 · DR 전략 고르기

웜 스탠바이는 보조 리전에 **작지만 완전히 동작하는** 복사본을 두고 장애 시 확장한다. 파일럿 라이트는 데이터 복제와 핵심만 켜 두고 나머지는 장애 시 **만들어야** 해서 RTO가 더 길다(10분 요구에서 흔한 함정). 다중 사이트는 예산 조건 위반이다.

📎 [DR 옵션(백서)](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)

**Q33 — 정답 D** · 고성능 · 같은 AZ 공유 블록 볼륨

EBS Multi-Attach는 **io1/io2**에서, **같은 AZ의 Nitro 인스턴스**에 한해 볼륨 하나를 여러 대에 붙인다. 조건(같은 AZ, 블록, 클러스터 파일 시스템)이 모두 맞을 때만 정답이 되는 보기다. gp3는 Multi-Attach를 지원하지 않고 EFS는 파일 스토리지다.

📎 [EBS Multi-Attach](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes-multi.html)

**Q34 — 정답 C** · 고성능 · 고정 IP + 빠른 리전 전환 API

Global Accelerator는 **고정 애니캐스트 IP**를 주고 상태 확인으로 리전을 빠르게 전환한다. CloudFront에도 별도 신청형 애니캐스트 고정 IP 기능이 있지만, 이 API는 캐시가 안 되는 동적 트래픽이고 **리전 엔드포인트 상태에 따른 빠른 전환**은 Global Accelerator의 대표 용도다. Route 53은 클라이언트 DNS 캐시 때문에 전환 시간이 불확실하고, ALB IP는 바뀐다.

📎 [Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)

**Q35 — 정답 A,E** · 복원력 · Route 53 액티브-패시브 (2개 선택)

장애 조치 라우팅은 **기본 레코드의 상태**를 보고 보조로 넘어간다. ALB 별칭 레코드는 대상 상태 평가로 ALB 뒤 대상의 상태를 반영할 수 있다. 가중치 라우팅은 평소에도 트래픽을 나눈다. Resolver 엔드포인트는 하이브리드 DNS용이다.

📎 [장애 조치 라우팅 구성](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-configuring.html)

**Q36 — 정답 C** · 복원력 · 장애 조치 중 연결 오류

RDS Proxy는 DB 상태를 직접 추적해 장애 조치 시 새 writer로 연결을 넘기고 클라이언트 연결을 유지해 **DNS 전파 지연을 우회**한다. 복제본 수·Global Database는 이 문제와 무관하고, 인스턴스 엔드포인트는 장애 조치 후 엉뚱한 인스턴스를 가리킨다.

📎 [RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)

**Q37 — 정답 C** · 보안 · SSH 전체 개방 자동 교정

Config 규칙이 **비준수를 감지**하고, 연결된 SSM Automation이 **자동으로 교정**한다. 설정 이력도 Config에 남는다. 보안 그룹 규칙 추가 자체를 막으면 정상 업무도 막힌다. cron 스크립트는 직접 운영해야 하고 실시간성이 떨어진다.

📎 [Config 자동 교정](https://docs.aws.amazon.com/config/latest/developerguide/remediation.html)

**Q38 — 정답 A** · 고성능 · S3로 Parquet 적재 — 운영 부담

모든 보기가 동작하지만, Firehose는 버퍼링·형식 변환·S3 전달을 **코드 없이** 해 준다. 재처리·다중 소비자가 필요 없다는 조건이 Kinesis Data Streams를 쓸 이유를 지운다.

📎 [Firehose 형식 변환](https://docs.aws.amazon.com/firehose/latest/dev/record-format-conversion.html)

**Q39 — 정답 B,D** · 고성능 · Athena 쿼리 비용·속도

Athena는 **스캔한 데이터 양**만큼 과금된다. 열 기반 형식은 필요한 열만 읽고 압축으로 양을 줄이며, 파티션은 필요한 날짜만 읽게 한다. Glue ETL이나 Firehose 형식 변환으로 Parquet을 만들 수 있다.

📎 [Athena 성능 튜닝](https://docs.aws.amazon.com/athena/latest/ug/performance-tuning.html)

**Q40 — 정답 D** · 고성능 · 대용량 업로드 한계

단일 PUT으로 올릴 수 있는 객체는 **최대 5GB**다. 그보다 크면 멀티파트 업로드가 필수다(객체 최대 크기는 그보다 훨씬 크다). Transfer Acceleration은 속도 기능이지 크기 한도를 바꾸지 않는다.

📎 [멀티파트 업로드](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)

</details>

<details>
<summary><b>Q41 ~ Q50 해설</b></summary>

**Q41 — 정답 B** · 보안 · 온프레미스 서버의 임시 자격 증명

IAM Roles Anywhere는 AWS 밖의 워크로드가 **X.509 인증서**로 임시 자격 증명을 받게 해 장기 액세스 키를 없앤다. Secrets Manager에서 키를 읽으려면 결국 그 API를 부를 자격 증명이 또 필요하다.

📎 [IAM Roles Anywhere](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html)

**Q42 — 정답 C** · 비용 최적화 · 다운로드 송신 비용

같은 파일이 반복 다운로드되면 CloudFront 캐시가 오리진 요청을 크게 줄이고, **S3 → CloudFront 구간 전송은 무료**다. Transfer Acceleration은 요금이 추가되고, 복제는 복제 전송·저장 비용이 늘며, One Zone-IA는 조회 요금 때문에 자주 읽는 데이터에 오히려 비싸다.

📎 [CloudFront 요금](https://aws.amazon.com/cloudfront/pricing/)

**Q43 — 정답 B,D** · 보안 · 교차 계정 SSE-KMS 객체 읽기 (2개 선택)

교차 계정 접근은 **양쪽 모두** 허용해야 한다. 계정 B의 IAM 정책은 B 쪽 허용이고, A 쪽에서는 **버킷 정책과 KMS 키 정책**이 B를 허용해야 한다. AWS 관리형 키(`aws/s3`)는 키 정책을 바꿀 수 없어 **교차 계정 공유가 안 되므로** 오히려 막힌다(그럴듯한 함정).

📎 [교차 계정 KMS 키 사용](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying-external-accounts.html)

**Q44 — 정답 C** · 비용 최적화 · 스냅숏 장기 보관

Snapshots Archive는 표준 계층보다 훨씬 싸게 보관하고(최소 90일, 복원에 24~72시간), 장기·드문 복원에 맞는다. 아카이브된 스냅숏은 **전체(full) 스냅숏**으로 저장되므로, 증분이 작은 스냅숏은 아카이브 이득이 적을 수 있다는 점도 기억하자.

📎 [EBS Snapshots Archive](https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-archive.html)

**Q45 — 정답 B** · 복원력 · EFS 비용과 복원력

One Zone은 데이터가 **AZ 하나에만** 있어 그 AZ가 장애 나면 접근할 수 없다. 백업은 복구 수단이지 "계속 접근"을 보장하지 않는다(그럴듯한 함정). 비용은 수명 주기 관리로 줄이는 것이 요구를 지키는 방법이다.

📎 [EFS 기능(스토리지 클래스)](https://docs.aws.amazon.com/efs/latest/ug/features.html) · [EFS 수명 주기 관리](https://docs.aws.amazon.com/efs/latest/ug/lifecycle-management-efs.html)

**Q46 — 정답 A** · 복원력 · SQS 중복 처리

처리 중인 메시지는 가시성 제한 시간이 지나면 **다시 보이게 되어** 다른 워커가 가져간다. 처리 시간보다 길게 잡는 것이 직접 원인 해결이다. FIFO는 중복 **전송**을 줄이지만 가시성 시간 초과로 인한 재처리는 여전히 생긴다(그럴듯한 함정). 표준 큐는 최소 1회 전송이므로 멱등 처리도 함께 갖추는 것이 좋다.

📎 [SQS 가시성 제한 시간](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)

**Q47 — 정답 D** · 비용 최적화 · 저장 비용이 큰 DynamoDB 테이블

Standard-IA 테이블 클래스는 **저장 단가가 낮고 읽기·쓰기 단가가 높아**, 저장 비용이 처리량 비용보다 큰 테이블에 맞는다. 테이블 단위 설정이라 코드 변경이 없다. 장기적으로는 TTL + S3 내보내기도 선택지다.

📎 [DynamoDB 테이블 클래스](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.TableClasses.html)

**Q48 — 정답 A** · 보안 · EKS Pod별 권한

Pod Identity·IRSA는 **서비스 계정 단위로 IAM 역할**을 매핑해 임시 자격 증명을 준다. 노드 그룹 분리도 동작은 하지만 노드 활용이 나빠지고 Pod 단위가 아니다. 액세스 키를 넣는 것은 장기 자격 증명이다.

📎 [EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html)

**Q49 — 정답 B** · 고성능 · Windows 공유 스토리지 성능

SMB + AD + Windows라는 조건 세 개가 FSx for Windows를 가리킨다. EFS는 NFS 전용이고, Lustre는 Linux 클라이언트 중심이다. File Gateway는 온프레미스용 하이브리드 서비스로 EC2 워크스테이션 간 고성능 공유의 답이 아니다.

📎 [FSx for Windows File Server](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)

**Q50 — 정답 D** · 보안 · 객체 수준 감사

CloudTrail은 기본적으로 **관리 이벤트**(버킷 생성·정책 변경 등)만 기록한다. 객체 단위 API(GetObject·PutObject·DeleteObject)는 **데이터 이벤트**로, 따로 켜야 하며 추가 요금이 있다. Config는 리소스 설정, 흐름 로그는 IP 패킷 메타데이터다.

📎 [CloudTrail 데이터 이벤트](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)

</details>

<details>
<summary><b>Q51 ~ Q60 해설</b></summary>

**Q51 — 정답 B** · 복원력 · 리전 간 DR — 비용과 RPO

리전 간 복제본은 비동기 복제로 보통 분 단위 이하 지연이라 RPO 5분을 만족하고, 승격은 수 분이면 된다. Aurora Global Database도 요구를 만족하지만 **엔진 이전과 비용**이 더 크다(가장 비용 효율적인 것이 아님). 매일 스냅숏은 RPO 24시간, Multi-AZ는 리전 장애를 막지 못한다.

📎 [리전 간 읽기 전용 복제본](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.XRgn.html)

**Q52 — 정답 C** · 복원력 · 고객별 순서 + 처리량

FIFO는 **같은 그룹 ID 안에서만** 순서를 지키며, 그룹 하나에 대해서는 한 번에 한 소비자만 진행한다. 그룹 ID를 주문별로 나누면 주문 내 순서를 지키면서 주문끼리 병렬 처리된다. 표준 큐는 순서 요구를 어긴다.

📎 [FIFO 메시지 그룹](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-key-terms.html) · [FIFO 높은 처리량](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/high-throughput-fifo.html)

**Q53 — 정답 B** · 보안 · 조직 전체 S3 퍼블릭 차단

계정 수준 퍼블릭 액세스 차단은 **계정의 모든 버킷**에 적용되고, SCP로 그 설정 변경을 막으면 관리자도 풀 수 없다. 버킷별 정책은 새 버킷마다 해야 하고, Config·Macie는 탐지다. (S3 퍼블릭 액세스 차단은 2023-04부터 **새 버킷 기본값**이기도 하다.)

📎 [S3 퍼블릭 액세스 차단](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)

**Q54 — 정답 C** · 보안 · RDS 비밀번호 교체 — 운영 부담

RDS는 마스터 비밀번호를 **Secrets Manager로 관리하는 통합 기능**이 있어 비밀 생성과 교체를 RDS가 맡는다. Lambda를 직접 작성하는 보기는 동작하지만 운영 부담이 더 크다. IAM DB 인증은 애플리케이션 사용자용으로 좋지만 마스터 비밀번호 교체 요구를 해결하지 않는다.

📎 [RDS와 Secrets Manager 비밀번호 관리](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-secrets-manager.html)

**Q55 — 정답 B** · 보안 · ALB 로그인 — 코드 최소

ALB는 OIDC·Cognito **인증 작업**을 리스너 규칙에서 지원해, 인증된 요청만 대상으로 넘기고 사용자 정보를 헤더로 전달한다. IP 제한은 "로그인한 직원" 요구를 충족하지 못하고(재택·VPN 등), 직접 구현은 코드 변경이 크다.

📎 [ALB 사용자 인증](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html)

**Q56 — 정답 B** · 복원력 · Direct Connect 최대 복원력

한 위치에 회선 두 개는 **위치 장애**에 약하다. DX + VPN 백업은 저비용 대안이지만 장애 시 대역폭이 떨어진다(조건 위반). AWS Direct Connect 복원력 권장 사항의 **최대 복원력** 모델은 여러 위치·여러 장비다.

📎 [Direct Connect 복원력 툴킷](https://docs.aws.amazon.com/directconnect/latest/UserGuide/resiliency_toolkit.html)

**Q57 — 정답 B** · 비용 최적화 · Spot 웹 계층

온디맨드 기준 용량 + 다양한 Spot 풀 + 용량 최적화 할당 + 재조정(회수 위험 신호 시 미리 교체)이 Spot을 안정적으로 쓰는 표준 구성이다. 단일 유형은 그 풀이 회수되면 한꺼번에 사라진다. 최고 가격을 올려도 **용량 부족에 따른 회수**는 막지 못한다.

📎 [혼합 인스턴스 그룹](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-mixed-instances-groups.html) · [용량 재조정](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-capacity-rebalancing.html)

**Q58 — 정답 A** · 복원력 · Kinesis 재처리 여유

Kinesis의 기본 보존 기간은 **24시간**이고 최대 365일까지 늘릴 수 있다. 샤드 수·팬아웃은 처리량·지연 문제이지 보존과 무관하다(그럴듯한 함정).

📎 [Kinesis 보존 기간 변경](https://docs.aws.amazon.com/streams/latest/dev/kinesis-extended-retention.html)

**Q59 — 정답 D** · 비용 최적화 · 메모리 기반 rightsizing

EC2 기본 지표에는 메모리가 없다. 에이전트로 메모리 지표를 보내면 Compute Optimizer가 이를 반영한다. 향상된 인프라 지표는 **관찰 기간을 늘리는** 옵션이라 메모리 지표 부재 문제를 해결하지 못한다(그럴듯한 함정).

📎 [Compute Optimizer 지표](https://docs.aws.amazon.com/compute-optimizer/latest/ug/metrics.html)

**Q60 — 정답 A** · 고성능 · 글로벌 저지연 쓰기

글로벌 테이블은 **각 리전에서 쓰기**가 가능한 다중 활성 복제를 제공한다. DAX·CloudFront는 읽기 캐시라 쓰기 지연을 줄이지 못한다. Aurora Global Database는 writer가 한 리전이라 다른 리전 쓰기는 멀리 간다.

📎 [DynamoDB 글로벌 테이블](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)

</details>

<details>
<summary><b>Q61 ~ Q65 해설</b></summary>

**Q61 — 정답 D** · 비용 최적화 · Aurora I/O 요금

I/O-Optimized는 I/O 요청 요금이 없고 인스턴스·스토리지 단가가 높아지는 구성으로, AWS 안내 기준 **I/O 비용이 Aurora 지출의 25%를 넘으면** 유리하다. Serverless v2는 용량 조절 문제이지 I/O 단가를 바꾸지 않는다. Aurora 스토리지는 EBS 볼륨 유형을 고르지 않는다.

📎 [Aurora 스토리지 구성](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html#aurora-storage-type)

**Q62 — 정답 A** · 비용 최적화 · 예산 초과 시 자동 차단

Budgets **예산 작업**은 IAM 정책·SCP 적용, EC2·RDS 중지를 자동(또는 승인 후)으로 실행한다. 이상 탐지는 고정 예산 초과 감지 도구가 아니고, 경보·보고서는 사람이 조치해야 한다.

📎 [Budgets 예산 작업](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-controls.html)

**Q63 — 정답 C** · 비용 최적화 · Lambda 비용 — 코드 변경 없이

arm64 Lambda는 x86보다 GB-초 단가가 낮다. 순수 Python 의존성이면 코드 변경 없이 옮길 수 있다. 메모리를 무작정 줄이면 CPU도 함께 줄어 실행 시간이 늘어 **오히려 비싸질 수** 있다(그럴듯한 함정). 프로비저닝된 동시성은 지연용이고 비용이 추가된다.

📎 [Lambda 명령 세트 아키텍처](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html)

**Q64 — 정답 B** · 보안 · 의도치 않은 외부 공유 찾기

IAM Access Analyzer는 리소스 기반 정책을 **논리적으로 분석**해 신뢰 영역(계정 또는 조직) 밖에 접근을 주는 리소스를 찾아 낸다. GuardDuty는 행위 기반 위협 탐지, Inspector는 취약점 스캔이다. Trusted Advisor도 일부 퍼블릭 버킷을 점검하지만 여러 리소스 유형의 교차 계정 접근 분석은 Access Analyzer의 일이다.

📎 [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)

**Q65 — 정답 D** · 비용 최적화 · 작은 객체 수억 개

Standard-IA·Glacier IR은 128KB 미만 객체도 **128KB로 과금**하므로 4KB 객체는 오히려 비싸지고, 수명 주기는 기본적으로 128KB 미만 객체를 전환하지도 않는다. Intelligent-Tiering도 128KB 미만 객체는 자동 이동 대상이 아니다. 객체 수가 많을수록 전환 **요청 요금**도 커진다. 묶어서 객체 수와 크기 문제를 함께 푸는 것이 정답이다.

📎 [수명 주기 전환 고려 사항](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html)

</details>


---

## 6. 채점과 결과 해석

### 6-1. 도메인별 점수표

| 도메인 | 문항 번호 | 문항 수 | 내 정답 수 | 정답률 |
|---|---|---|---|---|
| D1 안전한 아키텍처 (30%) | 7, 9, 10, 14, 15, 16, 23, 26, 28, 30, 31, 37, 41, 43, 48, 50, 53, 54, 55, 64 | 20 | ___ | ___ % |
| D2 복원력 있는 아키텍처 (26%) | 1, 2, 11, 12, 13, 17, 22, 24, 32, 35, 36, 45, 46, 51, 52, 56, 58 | 17 | ___ | ___ % |
| D3 고성능 아키텍처 (24%) | 3, 8, 18, 19, 20, 21, 25, 29, 33, 34, 38, 39, 40, 49, 60 | 15 | ___ | ___ % |
| D4 비용 최적화 아키텍처 (20%) | 4, 5, 6, 27, 42, 44, 47, 57, 59, 61, 62, 63, 65 | 13 | ___ | ___ % |
| **합계** | | **65** | ___ | ___ % |

W06의 도메인별 정답률을 옆에 적어 두고 비교해 보세요. **두 회차 모두 낮은 도메인**이 진짜 약점입니다.

| 도메인 | W06 정답률 | W07 정답률 | 판단 |
|---|---|---|---|
| D1 보안 | ___ % | ___ % | |
| D2 복원력 | ___ % | ___ % | |
| D3 고성능 | ___ % | ___ % | |
| D4 비용 | ___ % | ___ % | |

### 6-2. 점수대별 다음 행동

| 맞힌 개수(65문항) | 해석 | 다음 행동 |
|---|---|---|
| 52 이상 (80%+) | 함정까지 대비됨 | 시험 예약. 마지막 주에는 W05 치트시트와 이 편의 2절 함정 표만 반복 |
| 47~51 (72~79%) | 실전 합격권 | 틀린 문항의 **함정 유형**(2절 ①~⑦) 중 가장 많은 유형을 집중 복습, 1주 뒤 W06·W07 재응시 |
| 40~46 | 개념은 알지만 함정에 걸림 | 해설의 "그럴듯한 오답" 부분만 모아 노트로 정리, W04·W05 키워드 표를 다시 |
| 39 이하 | 기본 개념부터 | W06을 다시 80%까지 올린 뒤 W07에 재도전 |

### 6-3. 틀린 문항 → 다시 볼 편

| 약한 도메인 | 다시 볼 편 |
|---|---|
| D1 보안 | [W01 안전한 아키텍처](./w01-secure-architectures.md) — SCP와 IAM의 차이, 교차 계정 KMS, 엔드포인트·PrivateLink, 탐지 vs 예방 |
| D2 복원력 | [W02 복원력 있는 아키텍처](./w02-resilient-architectures.md) — DR 4단계의 RTO 차이, SQS 가시성 시간·FIFO 그룹, Route 53 장애 조치 |
| D3 고성능 | [W03 고성능 아키텍처](./w03-high-performing-architectures.md) — 캐시 키, S3 접두사·멀티파트, EBS 유형, NLB 리스너 |
| D4 비용 | [W04 비용 최적화](./w04-cost-optimization.md) — 약정 범위, S3 최소 과금, NAT·피어링·TGW 요금 구조 |
| 서비스 선택 실수 | [W05 서비스 비교 치트시트](./w05-service-comparison.md) |

> 💡 해설에서 "모르던 기능"이 나왔다면(예: Vault Lock, IAM Roles Anywhere, SnapStart) 클라우드 탭에서 직접 만들어 보세요. 보안 서비스는 [cloud09](../../cloud/cloud09-security.md), 비용 도구와 리소스 정리는 [cloud11](../../cloud/cloud11-finops-cleanup.md)에서 다룹니다.

---

## 자주 걸리는 함정 다시 보기

| 문항 주제 | 걸리기 쉬운 오답 | 왜 틀렸나 |
|---|---|---|
| CloudTrail 끄기 방지 | 관리자 그룹 IAM 정책 | 관리자는 자기 정책을 바꿀 수 있음 → SCP |
| 교차 계정 SSE-KMS | `aws/s3` 관리형 키로 재암호화 | AWS 관리형 키는 교차 계정 공유 불가 |
| 회원 전용 다운로드 | S3 presigned URL | 동작하지만 캐시가 없어 지연·비용 요구 미충족 |
| 리전 간 DR(RPO 5분) | Aurora Global Database | 요구보다 과하고 비쌈 → 리전 간 복제본 |
| RTO 10분 DR | 파일럿 라이트 | 장애 시 리소스를 만들어야 해 더 느림 → 웜 스탠바이 |
| 5일 승인 대기 | Step Functions Express | 최대 5분 → Standard |
| 세션 캐시 HA | Memcached 다중 노드 | 복제 없음 → Valkey/Redis OSS Multi-AZ |
| 4KB 객체 수억 개 | Standard-IA 전환 | 128KB 최소 과금 → 묶어서 저장 |
| RDS 3년 약정 | Compute SP / Database SP | Compute SP는 RDS 미적용, DB SP는 1년·할인 작음 → RDS RI |
| TLS 복호화 금지 | NLB TLS 리스너 | TLS 리스너도 종료함 → TCP 리스너 패스스루 |
| Java 콜드 스타트(추가 요금 없이) | 프로비저닝된 동시성 | 요금 추가 → SnapStart |

---

## 핵심 정리

- 함정 문항의 대부분은 **보기가 모두 동작**한다. 답은 질문 끝의 꼬리표(운영 부담·비용·코드 변경·추가 요금)와 시나리오 속 **조건 단어 하나**로 갈린다.
- "가장 비용 효율적인"은 **요구를 딱 만족하는 가장 싼 것**이지 가장 강한 것이 아니다(Global Database·io2·다중 사이트는 과한 정답이 되기 쉽다).
- 서비스 이름보다 **한도와 범위**를 외운다: Express 5분, SQS 지연 15분, Lambda 15분, 단일 PUT 5GB, IA 128KB·30일, Kinesis 기본 보존 24시간, EBS 기본 암호화는 리전 단위, CloudTrail 기본은 관리 이벤트만.
- 비용과 복원력이 부딪치면 **환경과 허용 중단**을 먼저 본다: 운영이면 AZ별 NAT·리전 EFS·이중화 DX, 개발이면 공유 NAT·One Zone이 정답이 될 수 있다.
- "막아야 한다"는 예방(SCP·정책·Vault Lock), "찾아야 한다"는 탐지(Config·GuardDuty·Access Analyzer·Macie)다.

---

## ⏭️ 다음 단계

SAA 시리즈는 여기까지입니다. W06·W07에서 안정적으로 합격권이 나왔다면 시험을 예약하고, 합격 후에는 다음 방향 중 하나를 골라 이어 가면 좋습니다(각 시험의 현재 코드·형식은 응시 전 AWS 공식 페이지에서 다시 확인하세요).

| 방향 | 자격증 | 이 시리즈와의 연결 | 이런 분께 |
|---|---|---|---|
| 설계 심화 | **AWS Certified Solutions Architect – Professional** | SAA의 네 도메인을 **다중 계정·조직 규모, 마이그레이션, 비용 거버넌스**로 넓힘. 문항이 길고 "여러 답이 모두 맞는" 유형이 W07보다 훨씬 많음 | 설계·아키텍처 리뷰를 맡는 분 |
| 운영 자동화 | **AWS Certified DevOps Engineer – Professional** | 본편 GitOps(Ch7)·관측성(Ch8)과 클라우드 탭 cloud05~08(Terraform·EKS·Argo CD·CloudWatch) | CI/CD·IaC·운영 자동화를 하는 분 |
| 네트워크 | **AWS Certified Advanced Networking – Specialty** | W01·W04·W05의 VPC·엔드포인트·DX·TGW·Route 53 Resolver를 깊게 | 하이브리드 네트워크·멀티 VPC 설계를 하는 분 |
| 보안 | **AWS Certified Security – Specialty** | 본편 보안 탭(Ch9), 클라우드 탭 cloud09, W07의 SCP·KMS·Access Analyzer·Network Firewall | 보안·컴플라이언스를 맡는 분 |
| 데이터·AI | **AWS Certified Data Engineer – Associate**, **AWS Certified Machine Learning Engineer – Associate** | W05의 Kinesis·Firehose·Athena·Glue, 본편 AI 인프라(Ch6)·클라우드 탭 cloud10(Bedrock vs GPU 노드) | 데이터 파이프라인·LLM 서빙을 다루는 분 |

> 💡 **추천 순서:** 실무가 설계 중심이면 **SAA → SAP**, 운영·플랫폼 중심이면 **SAA → (Terraform Associate) → DevOps Pro**, 보안 중심이면 **SAA → Security Specialty**. Kubernetes 쪽은 이 탭의 CKA·CKAD·CKS 시리즈와, IaC는 Terraform Associate 시리즈와 이어집니다.

자격증보다 중요한 것은 **손으로 만들어 본 경험**입니다. 클라우드 탭에서 VPC부터 EKS·관측성·보안·FinOps까지 실제 계정에 만들어 보고, 마지막에 [cloud11](../../cloud/cloud11-finops-cleanup.md)의 삭제 체크리스트로 깨끗이 정리하는 것까지 해 보세요. 그 경험이 SAP·Specialty 시험의 긴 시나리오를 읽는 힘이 됩니다.
