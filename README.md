# Cloud Security Study Guide

이 저장소는 `cloud security`를 체계적으로 공부하기 위한 개인 학습 공간이다. 목표는 서비스 이름을 많이 외우는 것이 아니라, 실제 클라우드 환경에서 어떤 보안 문제가 생기고 어떤 통제로 대응하는지 구조적으로 이해하는 데 있다.

## 1. Study Goal

- 클라우드 보안의 핵심 원칙을 이해한다.
- AWS, Azure, GCP에서 공통으로 반복되는 보안 개념을 먼저 익힌다.
- 실습을 통해 IAM, 네트워크, 로깅, 데이터 보호, 탐지/대응의 연결 구조를 익힌다.
- 단순 이론 정리가 아니라 "왜 필요한가 / 어떻게 설정하는가 / 무엇이 위험한가"까지 설명할 수 있는 수준을 목표로 한다.

## 2. Core Mindset

클라우드 보안은 기능 목록 공부가 아니라 책임 경계와 통제 지점을 파악하는 공부다. 항상 아래 질문으로 정리한다.

1. 누가 책임지는가?
2. 어떤 자산을 보호해야 하는가?
3. 어떤 공격 경로가 있는가?
4. 어떤 보안 통제가 이를 줄이는가?
5. 사고가 나면 어떤 로그로 탐지할 수 있는가?

## 3. Recommended Study Order

아래 순서로 공부한다. 앞 단계를 이해하지 못한 상태에서 뒤 단계로 가면 지식이 분절된다.

### Phase 1. Fundamentals

- Shared Responsibility Model
- IAM 기본 개념
- 클라우드 네트워크 기본
- 로그와 모니터링 기본
- 암호화와 키 관리 기본

### Phase 2. Identity And Access

- 사용자, 그룹, 역할, 정책
- Least Privilege
- MFA
- Federation / SSO
- Access Key 관리
- Privilege Escalation 패턴

### Phase 3. Network Security

- VPC / VNet 기본 구조
- Public vs Private Subnet
- Security Group / NSG / Firewall
- NACL 개념
- Bastion, VPN, Zero Trust 접근
- Load Balancer와 WAF

### Phase 4. Data Protection

- At Rest / In Transit 암호화
- KMS / Key Vault 개념
- Secret 관리
- Object Storage 보안 설정
- Database 접근 제어와 백업 보호
- Data Classification

### Phase 5. Logging, Detection, Response

- CloudTrail / Activity Log / Audit Log
- Config / Policy / Security Center류 서비스
- SIEM 연동 개념
- Alert 설계
- Incident Response 기본 흐름
- Forensics 관점의 로그 보존

### Phase 6. Governance And Architecture

- Multi-account / Multi-subscription 전략
- Organization Policy
- CSPM / CWPP / CIEM 개념
- DevSecOps 기초
- IaC 보안
- Compliance와 보안 기준선

## 4. Minimum Output For Each Topic

각 주제를 공부할 때 아래 5가지는 반드시 남긴다.

1. 한 줄 정의
2. 왜 중요한지
3. 대표적인 오해 또는 취약점
4. 설정 또는 운영 포인트
5. 관련 로그 / 탐지 포인트

예를 들어 `IAM Role`을 공부하면 단순히 "권한 위임 기능"이라고 끝내지 말고, 과도한 trust policy가 왜 위험한지와 어떤 이벤트로 추적할 수 있는지까지 적는다.

## 5. Weekly Study Loop

매주 아래 루프로 진행한다.

### Monday

- 이번 주 학습 주제 1~2개 선정
- 읽을 자료와 실습 목표 정의

### Tuesday To Thursday

- 개념 정리
- CLI 또는 콘솔 실습
- 실패 사례 기록

### Friday

- 이번 주 내용을 1페이지로 요약
- "내가 실제 운영자라면 어떤 설정을 기본값으로 둘지" 정리

### Weekend

- 복습
- 모르는 개념 다시 정리
- 다음 주 주제 선정

## 6. How To Study Effectively

아래 방식으로 공부해야 실제 실력이 남는다.

- 벤더 서비스 이름보다 보안 원리를 먼저 본다.
- 설정 화면을 보는 데서 끝내지 말고 공격 시나리오를 함께 본다.
- "허용/차단"만 보지 말고 "탐지/감사"까지 같이 본다.
- 실습 후 반드시 "잘못 설정하면 어떤 일이 생기는가"를 적는다.
- 하나의 개념을 AWS, Azure, GCP에서 비교해본다.

## 7. Priority Topics

처음 4주 동안은 아래 주제를 우선한다.

1. Shared Responsibility Model
2. IAM Policy / Role / Least Privilege
3. VPC, Subnet, Security Group
4. S3 / Object Storage 공개 설정 위험
5. CloudTrail 또는 감사 로그
6. KMS와 Secret 관리
7. Config / Policy 기반 통제
8. Incident Detection 기본 흐름

## 8. Lab Rules

실습은 아래 원칙으로 진행한다.

- 실습 목적을 먼저 적는다.
- 생성한 리소스와 권한을 기록한다.
- 예상 결과와 실제 결과를 비교한다.
- 비용이 발생할 수 있는 리소스는 종료 기준을 적는다.
- 실습 후 "어떤 보안 교훈을 얻었는가"를 남긴다.

## 9. Suggested Folder Structure

```text
.
├─ README.md
├─ docs/
│  ├─ fundamentals/
│  ├─ local-network/
│  ├─ domains/
│  ├─ aws/
│  ├─ azure/
│  └─ gcp/
├─ labs/
│  ├─ local/
│  ├─ aws/
│  ├─ azure/
│  ├─ gcp/
│  └─ reports/
├─ notes/
│  ├─ daily/
│  ├─ weekly-review/
│  └─ exam-prep/
├─ templates/
└─ scripts/
```

## 10. First 4 Weeks Plan

### Week 1

- Shared Responsibility Model
- IAM 기본
- Access Key와 MFA

### Week 2

- VPC / Subnet / Routing
- Security Group / NACL
- Public exposure 점검

### Week 3

- S3 / Blob / GCS 접근 제어
- Encryption at Rest / in Transit
- Secret 관리

### Week 4

- Audit Log
- Detection / Alert 기본
- Incident Response 초안

## 11. Definition Of Done

한 주제가 끝났다고 판단하려면 아래를 충족해야 한다.

- 개념을 3문장 안에 설명할 수 있다.
- 왜 위험한 설정이 되는지 예시를 들 수 있다.
- 최소 1개의 실습 결과를 남겼다.
- 관련 로그 또는 탐지 포인트를 말할 수 있다.
- 다른 클라우드에서도 비슷한 개념을 연결할 수 있다.

## 12. Next Actions

지금 바로 시작할 순서는 아래가 적절하다.

1. `docs/fundamentals/shared-responsibility.md` 작성
2. `docs/fundamentals/iam-basics.md` 작성
3. `labs/aws/iam/` 실습 시작
4. 첫 주 요약 노트 작성

## 13. Local Network Track

로컬 PC 웹서버 구축까지 이어질 과제는 아래 흐름으로 정리한다.

1. `docs/local-network/router-admin-survey.md`
2. `labs/local/router-admin/change-log.md`
3. `docs/local-network/local-web-server-roadmap.md`

문서와 실습 로그를 분리하는 이유는 다음과 같다.

- 조사 문서는 메뉴 구조와 기능 설명을 안정적으로 남기기 위함
- 실습 로그는 실제로 바꾼 설정, 결과, 복구 방법을 추적하기 위함
- 최종 목표 문서는 포트포워딩, DDNS, 방화벽, 웹서버 노출까지 연결하기 위함

---

이 저장소는 "기능 암기"가 아니라 "보안 구조 이해"를 목표로 운영한다. 공부할 때마다 항상 `자산 -> 권한 -> 노출면 -> 로그 -> 대응` 순서로 사고한다.
