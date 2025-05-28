# Kluis 생태계 설계 문서

## 🎯 전체 개요
**Kluis 생태계**는 VM 기반 애플리케이션 실행 환경을 제공하는 통합 솔루션입니다.  
**Kluis OS**(운영체제)와 **Kluis Manager**(VM 관리 도구)로 구성되며, 편의성을 최우선으로 설계되었습니다.

---

# 🖥️ Part 1: Kluis OS (운영체제)

## 개요
**Kluis OS**는 VM 친화적인 환경을 제공하는 Arch Linux 기반 운영체제입니다.  
기본적으로는 일반적인 데스크톱 OS이지만, VM 관리 도구와의 통합을 위한 최적화가 적용되어 있습니다.

## 핵심 구성요소

### 기반 시스템
- **베이스**: Arch Linux
- **데스크톱**: KDE Plasma
- **디스플레이**: Wayland
- **설치도구**: Anaconda installer

### 가상화 지원
- **하이퍼바이저**: QEMU/KVM (기본 설치)
- **라이브러리**: libvirt, SPICE
- **드라이버**: VirtIO 최적화

### 시스템 특징
- **경량화**: 불필요한 서비스 최소화
- **보안**: 기본 방화벽 및 격리 환경
- **성능**: VM 실행에 최적화된 커널 설정

## 설치 프로세스
1. **기본 설치**: Anaconda를 통한 표준 Arch Linux 설치
2. **데스크톱 구성**: KDE Plasma + Wayland 자동 설정
3. **가상화 준비**: QEMU/KVM, libvirt 설치 및 구성
4. **사용자 환경**: 기본 사용자 계정 및 권한 설정

## OS 레벨 기능
- **표준 데스크톱**: 일반적인 Linux 데스크톱 환경
- **가상화 최적화**: VM 실행을 위한 성능 튜닝
- **보안 기반**: 컨테이너화된 애플리케이션 실행 준비
- **확장성**: 추가적인 VM 관리 도구 설치 가능

---

# ⚙️ Part 2: Kluis Manager (VM 관리 도구)

## 개요
**Kluis Manager**는 Kluis OS(또는 다른 Linux 배포판) 위에서 동작하는 VM 관리 소프트웨어입니다.  
사용자 친화적인 VM 생성, 관리, 실행 환경을 제공합니다.

## 핵심 아키텍처

### 소프트웨어 구성
```
Kluis Manager Suite
├── Kluis Control Center (GUI 관리 도구)
├── kluis-cli (명령행 도구)  
├── Kluis Agent (VM 내부 에이전트)
├── Kluis Runtime (실행 환경)
└── Kluis Templates (VM 템플릿)
```

### VM 관리 방식
- **VM 명칭**: Kluis Instance
- **관리 단위**: 개별 VM들의 집합체
- **실행 모드**: 심리스/윈도우 모드 선택 가능

## 주요 기능

### 1. 유연한 심리스 모드
- **기본 상태**: 심리스 모드 활성화
- **일회성 변경**: `--windowed` 플래그로 독립 윈도우 실행
- **초기 설정**: 새 VM 생성 시 비심리스 모드에서 시작

### 2. 직관적 VM 관리
- VM 생성, 시작, 중지, 삭제
- 실시간 상태 모니터링
- 리소스 할당 및 관리

### 3. 데이터 통합
- **클립보드**: 호스트 ↔ VM 간 실시간 동기화
- **파일시스템**: 설정 가능한 공유 폴더
- **네트워크**: VM 간 선택적 연결

### 4. 애플리케이션 런처
- 호스트에서 VM 내 프로그램 직접 실행
- 애플리케이션별 VM 자동 선택
- 바로가기 및 메뉴 통합

## CLI 도구 - kluis

### 기본 VM 관리
```bash
# VM 라이프사이클
kluis create <name> [template]     # 새 VM 생성 (비심리스)
kluis start <name>                 # VM 시작 (심리스)
kluis start <name> --windowed      # 독립 윈도우로 시작
kluis stop <name>                  # VM 중지
kluis list                         # VM 목록
kluis delete <name>                # VM 삭제

# 애플리케이션 실행
kluis run <app> [vm]               # 심리스 모드 실행
kluis run <app> [vm] --windowed    # 독립 윈도우 실행
```

### 관리 기능
```bash
# 스냅샷 관리
kluis snapshot <vm> <name>         # 스냅샷 생성
kluis restore <vm> <name>          # 스냅샷 복원
kluis snapshots <vm>               # 스냅샷 목록

# 템플릿 관리
kluis template create <name> <vm>  # 템플릿 생성
kluis template list                # 템플릿 목록
kluis template clone <temp> <vm>   # 템플릿으로 VM 생성

# 설정 관리
kluis config seamless <vm> on/off  # 심리스 모드 기본값
kluis config share <vm> <path>     # 공유 폴더 설정
kluis config resources <vm> <cpu> <ram>  # 리소스 할당

# 모니터링
kluis status                       # 전체 상태
kluis monitor [vm]                 # 리소스 모니터링
```

## GUI 도구 - Kluis Control Center

### 메인 대시보드
- **VM 상태 패널**: 모든 VM 실행 상태 표시
- **시스템 리소스**: CPU, RAM, 디스크 사용량
- **빠른 액션**: VM 생성, 백업, 스냅샷

### VM 관리 패널
- **VM 카드 뷰**: 각 VM을 카드로 표시
- **실행 제어**: 시작/중지/재시작 버튼
- **모드 선택**: 심리스/윈도우 모드 토글
- **설정 관리**: 리소스, 네트워크, 공유 설정

### 애플리케이션 센터
- **통합 런처**: 모든 VM 앱을 통합 실행
- **앱 스토어**: VM별 애플리케이션 설치
- **바로가기 관리**: 데스크톱 통합 설정

### 백업 및 스냅샷
- **스냅샷 관리**: 시각적 타임라인 뷰
- **자동 백업**: 스케줄 설정 및 정책 관리
- **복원 도구**: 원클릭 롤백 기능

## 설치 및 구성

### Kluis Manager 설치
```bash
# Kluis OS에서 (기본 포함)
sudo pacman -S kluis-manager

# 다른 Linux 배포판에서
wget kluis-manager-installer.sh
sudo ./kluis-manager-installer.sh
```

### 초기 설정
1. **서비스 시작**: `systemctl enable kluis-manager`
2. **GUI 실행**: Kluis Control Center 시작
3. **첫 VM 생성**: 설정 마법사 실행
4. **환경 구성**: 공유 설정 및 템플릿 준비

## 사용자 워크플로우

### 일반 사용
1. **앱 실행**: 데스크톱 아이콘 → 자동으로 적절한 VM에서 실행
2. **모드 변경**: 우클릭 → "새 창에서 열기" (독립 윈도우)
3. **VM 관리**: Control Center에서 상태 확인 및 제어

### VM 생성 및 관리
1. **새 VM**: Control Center → "새 VM 생성" → 템플릿 선택
2. **초기 설정**: 비심리스 모드에서 OS 설치 및 구성
3. **운영 모드**: 설정 완료 후 심리스 모드로 전환

---

# 🔄 통합 운영 방식

## Kluis OS + Kluis Manager
- **기본 제공**: Kluis OS에는 Kluis Manager가 기본 설치
- **독립 설치**: 다른 Linux 배포판에도 Kluis Manager 설치 가능
- **완전 통합**: OS와 매니저가 함께 최적의 사용자 경험 제공

## 확장성
- **플러그인**: 추가 기능을 위한 확장 모듈
- **API**: 다른 도구와의 통합을 위한 REST API
- **템플릿**: 커뮤니티 기반 VM 템플릿 공유

## 호환성
- **OS 독립성**: Kluis Manager는 다양한 Linux에서 동작
- **VM 호환성**: 표준 QEMU/KVM 기반으로 높은 호환성
- **데이터 이식성**: VM 이미지 및 설정의 자유로운 이동
