# Multiboot Internals

## One-Line Definition

멀티부팅은 하나의 디스크에 여러 OS를 설치하고, 부트로더가 어떤 OS를 로딩할지 선택하게 하는 구조다.

## 부팅이란 무엇인가

전원을 켜면 OS가 실행되기까지 여러 단계를 거친다. 이 과정을 부팅이라고 한다.

크게 보면 아래 흐름이다.

```
전원 ON → 펌웨어 실행 → 부트로더 실행 → OS 커널 로딩 → OS 실행
```

펌웨어는 메인보드 칩에 새겨진 저수준 프로그램이다. OS가 없어도 실행된다. 이 펌웨어가 디스크에서 부트로더를 찾아 실행하고, 부트로더가 OS 커널을 메모리에 올린다.

멀티부팅을 이해하려면 이 흐름의 각 단계가 어떻게 동작하는지 알아야 한다.

## 두 가지 펌웨어: BIOS vs UEFI

### Legacy BIOS

오래된 방식이다. 1970년대에 설계된 구조가 2000년대 초반까지 쓰였다.

부팅 흐름:

```
전원 ON
  → BIOS 펌웨어 실행
  → POST(Power-On Self Test) 수행 — 하드웨어 초기화
  → 디스크 첫 512바이트(MBR) 읽음
  → MBR 안의 코드를 메모리에 올려 실행
  → 부트로더가 OS 커널 로딩
```

BIOS는 디스크에서 딱 첫 512바이트만 읽는다. 그 안에 부트로더 코드가 있어야 한다.

### UEFI

BIOS의 후계자다. 2000년대 중반부터 보급됐고 지금은 대부분의 PC가 UEFI를 쓴다.

부팅 흐름:

```
전원 ON
  → UEFI 펌웨어 실행
  → POST 수행
  → GPT 파티션 테이블 읽음
  → EFI System Partition 마운트
  → NVRAM에 저장된 부트 목록 순서대로 .efi 파일 실행
  → OS 커널 로딩
```

BIOS와 가장 큰 차이는 두 가지다.

첫째, UEFI는 파일시스템을 직접 읽을 수 있다. BIOS는 디스크의 첫 512바이트만 읽을 수 있었지만, UEFI는 FAT32 파티션을 통째로 읽고 그 안의 파일을 실행할 수 있다.

둘째, UEFI는 어떤 파일을 부팅할지 목록을 메인보드 내부(NVRAM)에 저장한다. 디스크가 없어도 이 목록은 유지된다.

## 파티션 테이블: MBR vs GPT

디스크를 어떻게 나눌지 기술하는 구조다. BIOS와 UEFI는 각각 다른 파티션 테이블 방식을 사용한다.

### MBR 파티션 테이블

BIOS와 함께 쓰는 구조다. 디스크의 첫 512바이트 안에 파티션 정보까지 함께 담는다.

```
디스크 첫 512바이트 (MBR)
├─ 0~445바이트: 부트로더 코드
├─ 446~509바이트: 파티션 테이블 (최대 4개 항목)
└─ 510~511바이트: 부팅 시그니처 (0x55AA)
```

한계:

- 파티션을 최대 4개까지만 만들 수 있다 (확장 파티션으로 우회하는 방법이 있지만 복잡하다)
- 디스크 크기 최대 2TB
- 파티션 테이블 백업이 없다 — 앞부분이 손상되면 복구가 어렵다

### GPT 파티션 테이블

UEFI와 함께 쓰는 구조다. MBR의 한계를 해결하기 위해 설계됐다.

```
디스크 구조
├─ Protective MBR (첫 512바이트)   — BIOS 하위호환용 더미
├─ GPT Header                       — 파티션 테이블 전체 정보 + 체크섬
├─ GPT Partition Entries            — 최대 128개 파티션 항목
│   각 항목: UUID, 시작 위치, 끝 위치, 파티션 타입, 이름
│
├─ [파티션 1] ...
├─ [파티션 2] ...
├─ ...
│
├─ GPT Partition Entries 백업
└─ GPT Header 백업                  — 디스크 맨 끝에 복사본
```

개선된 점:

- 파티션 최대 128개
- 디스크 크기 사실상 무제한 (최대 8ZB)
- 헤더와 파티션 테이블을 디스크 끝에 백업 — 앞부분이 손상돼도 복구 가능
- 각 파티션에 고유 UUID 부여

## EFI System Partition (ESP)

UEFI 부팅의 핵심 파티션이다.

UEFI 펌웨어가 직접 읽을 수 있는 FAT32 파티션으로, 부트로더 `.efi` 파일들이 여기 담긴다. 일반적으로 디스크의 첫 번째 파티션으로 만들며 크기는 100MB~1GB 정도다.

멀티부팅에서 Linux와 Windows는 이 파티션을 **공유**한다. 각자 자기 폴더만 사용하면 된다.

```
EFI System Partition (FAT32)
└─ EFI/
   ├─ ubuntu/                  ← Linux 부트로더 모음
   │  ├─ shimx64.efi
   │  ├─ grubx64.efi
   │  └─ grub.cfg
   │
   ├─ Microsoft/               ← Windows 부트로더 모음
   │  └─ Boot/
   │     ├─ bootmgfw.efi
   │     └─ BCD
   │
   └─ BOOT/
      └─ BOOTX64.EFI           ← 폴백 부트로더
```

## NVRAM 부트 목록

UEFI는 어떤 `.efi` 파일을 어떤 순서로 실행할지를 메인보드 내부의 **NVRAM**에 저장한다. 전원을 꺼도 이 정보는 유지된다.

Linux에서는 `efibootmgr` 명령으로 현재 목록을 볼 수 있다.

```
BootCurrent: 0002
BootOrder: 0001,0002,0003
Boot0001* Windows Boot Manager  → \EFI\Microsoft\Boot\bootmgfw.efi
Boot0002* Zorin OS              → \EFI\ubuntu\shimx64.efi
Boot0003* UEFI:Removable Device
```

UEFI 펌웨어는 전원이 켜지면 이 목록을 위에서부터 읽어 첫 번째로 유효한 항목을 실행한다.

메인보드의 BIOS/UEFI 설정 화면에서 이 순서를 바꿀 수 있고, `efibootmgr` 명령으로도 바꿀 수 있다.

## 부트로더

부트로더는 펌웨어와 OS 커널 사이를 잇는 프로그램이다. 역할은 OS 커널을 메모리에 올리고 제어권을 넘기는 것이다.

멀티부팅에서는 여러 OS를 선택할 수 있는 메뉴도 담당한다.

### GRUB (Linux)

Linux 진영의 표준 부트로더다. 대부분의 Linux 배포판이 기본으로 사용한다.

GRUB은 크게 두 단계로 동작한다.

**1단계** — UEFI가 `shimx64.efi` → `grubx64.efi`를 실행한다.

**2단계** — GRUB이 `/boot/grub/grub.cfg`를 읽어 메뉴를 구성하고 표시한다.

```
┌──────────────────────────────────┐
│  Zorin OS                        │
│  Zorin OS (recovery mode)        │
│  Windows Boot Manager            │
└──────────────────────────────────┘
```

사용자가 Linux를 선택하면 GRUB이 커널 파일(`vmlinuz`)을 직접 메모리에 올린다. Windows를 선택하면 GRUB은 Windows Boot Manager에게 제어권을 넘긴다. 이를 **체인로딩(chainloading)** 이라고 한다.

GRUB 설정은 `/boot/grub/grub.cfg`에 있다. `update-grub` 명령을 실행하면 디스크에 설치된 OS를 자동으로 탐지해서 이 파일을 다시 만든다.

### Windows Boot Manager

Windows 전용 부트로더다. ESP 안의 `bootmgfw.efi`가 실체다.

```
Windows Boot Manager (bootmgfw.efi)
  → BCD 읽음
  → winload.efi 실행
  → ntoskrnl.exe (Windows 커널) 메모리에 로딩
  → Windows 실행
```

BCD(Boot Configuration Data)는 어느 드라이브, 어느 파티션에서 Windows를 로딩할지 담은 데이터베이스다.

## Linux 커널 로딩 과정

GRUB이 Linux를 선택했을 때 실제로 어떤 일이 일어나는지다.

```
GRUB
  → vmlinuz (압축된 커널 이미지) 메모리에 로딩
  → initramfs (초기 램디스크) 메모리에 로딩
  → 커널 압축 해제 및 실행
  → initramfs 안의 임시 파일시스템 마운트
  → 실제 루트 파티션 마운트
  → initramfs 언마운트
  → /sbin/init (또는 systemd) 실행
  → 서비스들 순차 실행
  → 로그인 화면
```

**initramfs**가 필요한 이유가 있다. 커널은 처음 실행될 때 루트 파티션을 마운트해야 하는데, 그러려면 파일시스템 드라이버가 필요하다. 그런데 그 드라이버는 파일시스템 안에 있다. 닭이 먼저냐 달걀이 먼저냐 문제다. initramfs는 이 문제를 해결하기 위한 임시 최소 파일시스템이다. 커널이 initramfs를 먼저 램에 올린 뒤, 그 안의 드라이버로 진짜 루트 파티션을 마운트한다.

## Secure Boot와 shim

Secure Boot는 UEFI의 기능 중 하나다. `.efi` 파일을 실행하기 전에 디지털 서명을 검증한다. 서명이 없거나 신뢰할 수 없는 서명이면 실행을 거부한다.

기본 신뢰 키는 Microsoft 키다. PC 제조사들이 출하할 때 Microsoft 키를 넣어두기 때문이다.

여기서 문제가 생긴다. GRUB은 Microsoft 서명이 없다.

해결책이 **shim**이다. shim은 Canonical(Ubuntu 제작사)이 만들고 Microsoft가 서명해준 작은 프로그램이다. shim 자체는 Microsoft 서명이 있어서 UEFI가 허용한다. shim이 실행되면 그다음에 GRUB을 Canonical 키로 검증하고 실행한다.

```
UEFI
  → shimx64.efi 실행  (Microsoft 서명 → UEFI가 허용)
    → grubx64.efi 실행  (Canonical 서명 → shim이 허용)
      → vmlinuz 실행  (Canonical 서명 → GRUB이 허용)
```

Windows는 처음부터 Microsoft 서명이 있어서 shim 없이 바로 실행된다.

## 전체 흐름 정리

```
전원 ON
│
▼
UEFI 펌웨어
  NVRAM에서 BootOrder 읽음
  첫 번째 유효한 항목 실행
│
▼
GRUB (shimx64.efi → grubx64.efi)
  grub.cfg 읽어서 메뉴 구성
  사용자 선택 대기
│
├─ Linux 선택
│    vmlinuz + initramfs 로딩
│    → 커널 실행
│    → initramfs로 루트 파티션 마운트
│    → systemd 실행
│    → Linux 부팅 완료
│
└─ Windows 선택
     bootmgfw.efi로 체인로딩
     → BCD 읽음
     → winload.efi
     → ntoskrnl.exe
     → Windows 부팅 완료
```

## 설치 순서가 중요한 이유

Windows는 설치할 때 NVRAM의 BootOrder를 바꿔서 자신을 맨 앞에 등록한다.

그래서 순서에 따라 결과가 달라진다.

**Windows 먼저, Linux 나중 (권장)**

Linux를 설치할 때 GRUB이 디스크를 스캔해 Windows를 자동으로 찾는다. GRUB 메뉴에 Windows가 자동으로 추가된다. 그리고 GRUB을 BootOrder 맨 앞에 등록한다. 정상 동작한다.

**Linux 먼저, Windows 나중 (문제 발생)**

Windows 설치 후 BootOrder가 바뀐다. Windows Boot Manager가 맨 앞에 올라서 GRUB이 실행되지 않는다. 아래 명령으로 복구할 수 있다.

```bash
# Linux Live USB로 부팅 후
sudo grub-install /dev/nvme0n1
sudo update-grub
```

## Common Misunderstandings

- "멀티부팅하면 두 OS가 동시에 실행된다"
  아니다. 한 번에 하나의 OS만 실행된다. 부팅할 때 하나를 선택하면 그것만 실행된다.

- "GRUB이 Windows를 직접 부팅한다"
  아니다. GRUB은 Windows Boot Manager에게 제어권을 넘길 뿐이다. 실제 Windows 부팅은 Windows Boot Manager가 담당한다.

- "ESP는 Linux 전용 파티션이다"
  아니다. Linux와 Windows가 공유한다. 각자 자기 폴더만 쓴다.

- "Secure Boot를 끄지 않으면 Linux를 설치할 수 없다"
  아니다. shim 덕분에 Secure Boot가 켜진 상태에서도 대부분의 Linux 배포판을 설치하고 부팅할 수 있다.

## Q&A

### POST가 뭔가

Power-On Self Test다. 전원이 켜지는 순간 펌웨어가 CPU, RAM, 그래픽카드 등 하드웨어가 정상인지 스스로 점검하는 과정이다. 이상이 있으면 비프음이나 에러 화면으로 알려준다. 정상이면 부팅으로 넘어간다. 부팅과 별개 단계지만 항상 먼저 일어난다.

### MBR과 GPT가 뭔가

둘 다 "디스크를 어떻게 나눴는지 기록하는 구조"다. 디스크 자체는 그냥 빈 공간이기 때문에, 어디서 어디까지가 어떤 파티션인지 어딘가에 기록해둬야 한다. 그 기록 방식이 MBR이냐 GPT냐의 차이다.

- **MBR**: 디스크 맨 앞 512바이트에 파티션 정보와 부트 코드를 함께 구겨 넣은 구형 방식
- **GPT**: 디스크 앞뒤에 파티션 정보를 따로 저장하는 현대 방식. 백업도 있고 파티션 수 제한도 없다

현재 이 시스템의 디스크는 GPT다.

### Secure Boot는 언제 사용하는건가

항상 켜져 있는 게 기본이다. 껐다 켰다 선택하는 기능이라기보다, 메인보드 출고 시 기본값이 ON이다.

역할은 하나다 — 부트로더가 서명된 것인지 확인한다. 서명 없는 `.efi` 파일은 실행 자체를 막는다.

일반 사용자 입장에서 직접 신경 쓸 일은 별로 없다. Ubuntu/Zorin 같은 메이저 배포판은 shim이 포함되어 있어서 Secure Boot가 켜진 상태로도 그냥 설치되고 부팅된다. 직접 커스텀 커널을 빌드하거나 서명 없는 부트로더를 쓸 때 Secure Boot를 끄거나 직접 키를 등록하는 작업이 필요해진다.

### Linux가 메인인데 Windows를 추가 설치할 때 Windows에 우선권을 줘야 하는가

줄 필요 없다. 오히려 주면 불편해진다.

Windows를 나중에 설치하면 Windows가 자동으로 자기 자신을 BootOrder 맨 앞에 올린다. 그러면 전원 켤 때마다 GRUB 메뉴 없이 바로 Windows로 부팅된다.

그래서 Windows 설치 후에 아래 두 가지를 해줘야 한다.

```bash
# GRUB이 Windows를 인식하도록 갱신
sudo update-grub

# GRUB을 BootOrder 맨 앞으로 복구
sudo grub-install /dev/nvme0n1
```

이러면 다시 GRUB 메뉴가 먼저 뜨고, 거기서 Linux 또는 Windows를 선택할 수 있다.

### Windows Boot Manager가 아니라 GRUB이 먼저 뜨게 하려면

NVRAM의 BootOrder에서 GRUB 항목을 Windows Boot Manager보다 앞에 두면 된다.

```bash
# 현재 상태 확인
efibootmgr

# 출력 예시
# Boot0001* Windows Boot Manager
# Boot0002* Zorin OS
# BootOrder: 0001,0002   ← Windows가 앞에 있는 상태

# GRUB을 맨 앞으로 변경
sudo efibootmgr --bootorder 0002,0001
```

또는 메인보드 UEFI 설정 화면(부팅 시 F2/Del 등)에서 Boot Priority 항목을 직접 바꿔도 된다.

GRUB이 먼저 실행되면, GRUB 메뉴에서 Linux를 선택하든 Windows를 선택하든 자유다. Windows를 선택하면 GRUB이 Windows Boot Manager를 체인로딩해준다.

### 부트 파티션만 나누고 내부 파티션은 공유 가능한가

"공유"의 의미에 따라 다르다.

파티션 자체를 두 OS가 동시에 마운트해서 쓰는 건 불가능하다. 한 번에 하나의 OS만 실행되기 때문에 물리적으로 동시에 쓰는 상황은 생기지 않지만, 파일시스템 충돌 위험이 있어서 권장하지 않는다.

데이터 파티션을 별도로 만들어서 두 OS가 번갈아 접근하는 건 가능하다. FAT32나 NTFS로 포맷한 파티션을 만들면 Linux에서도 마운트하고 Windows에서도 읽을 수 있다.

```
nvme0n1
├─ p1: EFI System Partition (FAT32, 512MB)    ← 부트로더 공유
├─ p2: Linux root (ext4)                       ← Linux 전용
├─ p3: Windows (NTFS)                          ← Windows 전용
└─ p4: 공유 데이터 (FAT32 또는 NTFS)          ← 양쪽에서 접근 가능
```

주의할 점은 파일시스템 호환성이다.

- Linux의 루트 파티션은 ext4인데, Windows는 ext4를 기본으로 읽지 못한다. 서드파티 드라이버를 설치하면 가능하다.
- Linux에서 NTFS 파티션을 읽고 쓰는 것은 기본 지원된다.
- 양쪽 모두에서 접근하는 공유 파티션을 만든다면 FAT32 또는 NTFS가 현실적인 선택이다.
