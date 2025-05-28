# Kluis OS 운영체제 설계 문서
## 🎯 개요
**Kluis OS**는 모든 애플리케이션을 가상머신(Kluis) 내에서 실행하는 혁신적인 운영체제입니다.  
보안보다는 **편의성을 최우선**으로 하며, 사용자에게 투명한 가상화 환경을 제공합니다.

---

## 🏗️ 핵심 아키텍처
### 호스트 시스템
- **기반 OS**: Arch Linux
- **데스크톱 환경**: KDE Plasma
- **디스플레이 서버**: Wayland
- **설치 도구**: Anaconda installer

### 가상화 환경
- **VM 엔진**: QEMU/KVM
- **VM 명칭**: Kluis
- **관리 도구**: Kluis Manager
- **실행 방식**: 모든 사용자 애플리케이션이 Kluis VM 내에서 실행

---

## ✨ 핵심 기능
### 1. 심리스 모드 (기본 활성화)
- VM과 호스트 간 경계가 보이지 않는 통합 환경
- 윈도우가 자연스럽게 호스트 데스크톱에 통합

### 2. 클립보드 공유
- **호스트 ↔ Kluis VM** 간 실시간 동기화
- **Kluis VM ↔ Kluis VM** 간 상호 공유

### 3. 파일 공유
- **호스트 ↔ Kluis VM** 간 파일시스템 공유
- **Kluis VM ↔ Kluis VM** 간 데이터 교환

### 4. 통합 실행 환경
- 호스트 데스크톱의 바로가기로 VM 내 프로그램 직접 실행
- 사용자는 VM 존재를 의식하지 않고 사용

---

## 🔧 시스템 구조
### 호스트 레벨
```
Kluis OS (Arch Linux 기반)
├── KDE Plasma (Wayland)
├── Kluis Manager (VM 관리 도구)
├── 최소한의 시스템 서비스
└── Kluis VM Runtime Environment
```

### VM 레벨
```
Kluis VM Instances
├── 사용자 애플리케이션들
├── Kluis Agent (호스트 통신 담당)
├── 공유 서비스 (클립보드, 파일시스템)
└── 경량화된 게스트 OS
```

---

## 🛠️ 주요 컴포넌트
### Kluis Manager
- VM 생성, 삭제, 관리
- 리소스 할당 및 제어
- 네트워크 설정 관리
- 스냅샷 및 백업 기능

### Kluis Agent
- 호스트-게스트 간 통신 브릿지
- 파일 및 클립보드 실시간 동기화
- 심리스 모드 구현
- 애플리케이션 바로가기 연동

### 통합 런처
- 호스트 데스크톱의 애플리케이션 아이콘
- VM 내 프로그램의 투명한 실행
- 윈도우 관리 시스템 통합

---

## 💻 CLI 도구 - kluis-cli
### 기본 명령어 구조
```bash
# VM 관리
kluis vm create <name> [template]     # 새 VM 생성
kluis vm list                         # VM 목록 조회
kluis vm start <name>                 # VM 시작
kluis vm stop <name>                  # VM 중지
kluis vm delete <name>                # VM 삭제
kluis vm status <name>                # VM 상태 확인

# 템플릿 관리
kluis template list                   # 템플릿 목록
kluis template create <name> <vm>     # VM에서 템플릿 생성
kluis template clone <template> <vm>  # 템플릿으로부터 VM 복제

# 스냅샷 관리
kluis snapshot create <vm> <name>     # 스냅샷 생성
kluis snapshot list <vm>              # 스냅샷 목록
kluis snapshot restore <vm> <name>    # 스냅샷 복원
kluis snapshot delete <vm> <name>     # 스냅샷 삭제

# 애플리케이션 관리
kluis app install <package> [vm]      # VM에 애플리케이션 설치
kluis app launch <app> [vm]           # 애플리케이션 실행
kluis app list [vm]                   # 설치된 앱 목록

# 시스템 모니터링
kluis monitor resources               # 리소스 사용량 모니터링
kluis monitor processes <vm>          # VM 프로세스 모니터링
kluis monitor network                 # 네트워크 상태 확인

# 설정 관리
kluis config set <key> <value>        # 설정 변경
kluis config get <key>                # 설정 조회
kluis config reset                    # 설정 초기화
```

### 고급 명령어
```bash
# 네트워크 관리
kluis network create <name> <cidr>    # 가상 네트워크 생성
kluis network attach <vm> <network>   # VM을 네트워크에 연결
kluis network isolate <vm>            # VM 네트워크 격리

# 파일 공유 관리
kluis share add <path> <vm>           # 공유 폴더 추가
kluis share remove <path> <vm>        # 공유 폴더 제거
kluis share list <vm>                 # 공유 폴더 목록

# 로그 및 디버깅
kluis log show <vm>                   # VM 로그 조회
kluis log export <vm> <file>          # 로그 내보내기
kluis debug <vm>                      # 디버그 모드 접속

# 백업 및 복구
kluis backup create <vm> <path>       # VM 백업
kluis backup restore <path> <vm>      # 백업 복원
kluis backup list                     # 백업 목록

# 성능 튜닝
kluis optimize <vm>                   # VM 성능 최적화
kluis resources allocate <vm> <cpu> <ram>  # 리소스 할당 조정
```

### 편의 기능
```bash
# 원클릭 명령어
kluis quick-setup                     # 초기 환경 자동 구성
kluis quick-backup                    # 모든 VM 자동 백업
kluis quick-update                    # 모든 VM 업데이트

# 배치 작업
kluis batch start-all                 # 모든 VM 시작
kluis batch stop-all                  # 모든 VM 중지
kluis batch snapshot-all              # 모든 VM 스냅샷 생성

# 자동화 스크립트
kluis script record <name>            # 작업 스크립트 녹화
kluis script play <name>              # 스크립트 실행
kluis script list                     # 스크립트 목록
```

---

## 🖥️ GUI 관리 도구 - Kluis Control Center
### 메인 대시보드
**시스템 개요 화면**
- 실행 중인 VM 수 및 상태 표시
- 전체 시스템 리소스 사용량 (CPU, RAM, 디스크)
- 최근 활동 로그 (VM 시작/종료, 앱 실행)
- 빠른 액션 버튼 (새 VM 생성, 전체 백업, 시스템 최적화)

**실시간 모니터링 위젯**
- VM별 리소스 사용량 그래프
- 네트워크 트래픽 모니터링
- 디스크 I/O 현황
- 성능 알림 및 경고

### VM 관리 패널
**VM 그리드 뷰**
- 썸네일과 함께 VM 목록 표시
- 각 VM의 상태 (실행중/중지/일시정지)
- 우클릭 컨텍스트 메뉴 (시작/중지/설정/삭제)
- 드래그 앤 드롭으로 VM 정렬

**VM 상세 설정**
- 리소스 할당 슬라이더 (CPU 코어, RAM, 디스크)
- 네트워크 어댑터 설정
- 공유 폴더 관리
- 디스플레이 설정 (해상도, 멀티 모니터)
- 부팅 순서 및 옵션

**VM 생성 마법사**
- 단계별 안내 (템플릿 선택 → 리소스 할당 → 네트워크 설정)
- 사전 구성된 템플릿 (개발용, 게임용, 오피스용)
- 실시간 미리보기
- 자동 최적화 옵션

### 애플리케이션 관리
**애플리케이션 스토어**
- 카테고리별 애플리케이션 브라우징
- 인기/추천 앱 섹션
- 원클릭 설치 (자동으로 적절한 VM 선택)
- 리뷰 및 평점 시스템

**런처 관리**
- 데스크톱 바로가기 편집기
- 애플리케이션 아이콘 커스터마이징
- 시작 메뉴 구성
- 즐겨찾기 및 최근 사용 앱

**애플리케이션 배포**
- VM 간 애플리케이션 복사/이동
- 설정 프로파일 내보내기/가져오기
- 자동 업데이트 관리

### 스냅샷 및 백업 센터
**시각적 스냅샷 관리**
- 타임라인 뷰로 스냅샷 히스토리 표시
- 썸네일과 메모가 포함된 스냅샷 미리보기
- 브랜치 형태로 스냅샷 트리 시각화
- 드래그 앤 드롭 복원

**자동 백업 스케줄러**
- 시각적 스케줄 설정 (달력 인터페이스)
- 백업 정책 템플릿 (일일/주간/월간)
- 클라우드 백업 통합 (Google Drive, Dropbox)
- 백업 상태 알림

### 네트워크 구성 도구
**네트워크 토폴로지 뷰**
- VM 간 네트워크 연결을 그래프로 시각화
- 드래그 앤 드롭으로 네트워크 설정
- 실시간 트래픽 플로우 표시
- 방화벽 규칙 시각적 편집

**보안 정책 관리**
- VM 간 통신 규칙 설정
- 인터넷 접근 제어
- 포트 포워딩 관리
- VPN 연결 설정

### 설정 및 환경설정
**사용자 인터페이스 설정**
- 테마 및 색상 스키마 선택
- 대시보드 위젯 커스터마이징
- 단축키 설정
- 알림 및 사운드 설정

**시스템 최적화**
- 자동 최적화 옵션
- 리소스 할당 전략 선택
- 성능 프로파일 (절전/균형/고성능)
- 시스템 클린업 도구

**고급 설정**
- Kluis Agent 구성
- 심리스 모드 세부 옵션
- 로그 레벨 및 디버깅 설정
- 플러그인 및 확장 관리

### 통합 기능
**파일 관리자 통합**
- 호스트와 VM 파일시스템 통합 뷰
- 드래그 앤 드롭 파일 전송
- 공유 폴더 실시간 동기화 상태
- 파일 버전 관리

**작업 관리자**
- 전체 시스템 프로세스 뷰 (호스트 + 모든 VM)
- VM별 프로세스 분류
- 리소스 사용량 정렬 및 필터링
- 프로세스 강제 종료 및 우선순위 조정

---

## 📦 설치 프로세스
### 1. 기본 시스템 설치
- Anaconda를 통한 Arch Linux 기반 설치
- KDE Plasma + Wayland 자동 구성

### 2. Kluis 환경 구성
- 가상화 도구 설치 (QEMU/KVM, libvirt)
- Kluis Manager 및 Agent 설치
- SPICE/VirtIO 드라이버 설정

### 3. 초기 설정
- 기본 Kluis VM 템플릿 생성
- 사용자 환경 초기화
- 애플리케이션 바로가기 설정

---

## 🎮 사용자 워크플로우
### 부팅 및 로그인
1. Kluis OS 부팅
2. KDE Plasma 로그인
3. Kluis Manager 자동 시작

### 애플리케이션 사용
1. 데스크톱 아이콘 클릭
2. 자동으로 적절한 Kluis VM에서 실행
3. 심리스 모드로 투명한 사용 경험

### 파일 및 데이터 작업
1. 호스트-VM 간 자동 파일 동기화
2. 클립보드 내용 실시간 공유
3. 여러 Kluis VM 간 데이터 교환

### 시스템 종료
1. 모든 VM 상태 자동 저장
2. 안전한 시스템 종료

---

## 🚀 편의성 중심 설계
### 투명성
- 사용자는 VM 존재를 의식하지 않음
- 기존 데스크톱과 동일한 사용 경험

### 자동화
- VM 리소스 자동 할당 및 관리
- 백그라운드에서 투명한 상태 관리

### 통합성
- 호스트와 VM 간 완전한 통합 환경
- 원클릭으로 모든 작업 수행

---

## 🔒 기본 보안 기능
### 격리
- 호스트 시스템과 애플리케이션 완전 분리
- VM 간 선택적 네트워크 격리

### 백업 및 복구
- 스냅샷 기반 즉시 롤백
- 자동 백업 시스템

### 제한된 호스트 접근
- 호스트에 직접 설치/실행 제한
- 모든 작업은 Kluis VM을 통해서만 수행****
