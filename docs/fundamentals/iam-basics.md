# IAM Basics

## One-Line Definition

IAM(Identity and Access Management)은 누가 어떤 리소스에 어떤 조건으로 접근할 수 있는지를 통제하는 체계다.

## Why IAM Comes First

클라우드 보안에서 가장 먼저 봐야 할 것은 권한이다. 공격자는 취약점만 노리는 것이 아니라, 과한 권한, 노출된 액세스 키, 잘못된 trust policy를 이용해 가장 쉽게 확장한다.

## Core Concepts

### User

사람이나 애플리케이션을 위한 개별 계정이다. 장기 자격증명(예: access key)을 가질 수 있어 관리에 특히 주의해야 한다.

### Group

여러 사용자에게 공통 권한을 묶어서 부여하기 위한 단위다. 직접 로그인 주체는 아니고 권한 관리 편의를 위한 구조다.

### Role

특정 권한 세트를 일시적으로 위임받기 위한 정체성이다. 보통 사람, 서비스, 다른 계정, 워크로드가 역할을 assume 해서 사용한다.

### Policy

무엇을 허용하거나 거부할지 정의한 규칙 문서다. 어떤 액션을 어떤 리소스에 대해 수행할 수 있는지 결정한다.

## Important Principles

### Least Privilege

필요한 최소 권한만 부여한다. `AdministratorAccess`를 습관적으로 붙이는 운영은 가장 흔한 보안 부채다.

### MFA

관리 콘솔 접근에는 MFA를 기본값으로 봐야 한다. 비밀번호 유출만으로 계정이 탈취되지 않도록 하는 최소 통제다.

### Temporary Credentials

가능하면 장기 access key보다 role 기반 임시 자격증명을 선호한다.

### Separation Of Duties

운영, 보안, 감사 권한을 필요 시 분리한다. 한 계정이 모든 행위를 할 수 있으면 사고 조사와 통제가 무너진다.

## High-Risk Patterns

- 모든 사용자에게 광범위한 관리자 권한 부여
- 장기 access key를 오래 유지
- 퇴사자/미사용 계정 방치
- trust policy를 과하게 열어둠
- root 계정 일상 사용
- MFA 미적용

## User vs Role

처음에는 이 차이를 확실히 잡아야 한다.

- User: 장기적으로 존재하는 계정
- Role: 필요할 때 위임받아 쓰는 권한 단위

실무에서는 사람에게 직접 강한 권한을 계속 주기보다, 기본 계정은 약하게 두고 필요한 순간 role을 통해 승격하는 방식이 더 안전하다.

## Operational Checklist

- 루트 계정에 MFA가 적용되어 있는가
- 관리자 권한이 꼭 필요한 사용자만 가지는가
- access key가 장기 방치되고 있지 않은가
- 서비스 간 접근은 role 기반으로 설계되었는가
- 누가 어떤 정책을 받았는지 설명 가능한가

## Detection Angle

IAM은 반드시 로그와 같이 봐야 한다.

- 누가 로그인했는가
- 누가 정책을 바꿨는가
- 누가 role을 assume 했는가
- 누가 access key를 생성했는가
- 실패한 인증 시도가 반복되는가

## Day 1 Check

아래를 설명할 수 있으면 다음 단계로 넘어가도 된다.

- User, Group, Role, Policy 차이
- Least Privilege가 왜 중요한지
- 장기 access key보다 role이 더 안전한 이유
- MFA가 없는 관리자 계정이 왜 위험한지
