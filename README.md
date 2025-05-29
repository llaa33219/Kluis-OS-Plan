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
- **GPU 가상화**: VFIO, GPU 패스스루 지원

### 시스템 특징
- **경량화**: 불필요한 서비스 최소화
- **보안**: 기본 방화벽 및 격리 환경
- **성능**: VM 실행에 최적화된 커널 설정
- **GPU 최적화**: GPU 리소스 동적 할당 및 스케줄링

## 설치 프로세스
1. **기본 설치**: Anaconda를 통한 표준 Arch Linux 설치
2. **데스크톱 구성**: KDE Plasma + Wayland 자동 설정
3. **가상화 준비**: QEMU/KVM, libvirt 설치 및 구성
4. **GPU 설정**: VFIO 및 GPU 패스스루 자동 구성
5. **사용자 환경**: 기본 사용자 계정 및 권한 설정

## OS 레벨 기능
- **표준 데스크톱**: 일반적인 Linux 데스크톱 환경
- **가상화 최적화**: VM 실행을 위한 성능 튜닝
- **GPU 리소스 관리**: 동적 GPU 할당 및 스케줄링
- **보안 기반**: 컨테이너화된 애플리케이션 실행 준비
- **확장성**: 추가적인 VM 관리 도구 설치 가능

---

# ⚙️ Part 2: Kluis Manager (VM 관리 도구)

## 개요
**Kluis Manager**는 Kluis OS(또는 다른 Linux 배포판) 위에서 동작하는 VM 관리 소프트웨어입니다.  
사용자 친화적인 VM 생성, 관리, 실행 환경을 제공하며, 고급 GPU 리소스 공유 기능을 포함합니다.

## 핵심 아키텍처

### 소프트웨어 구성
```
Kluis Manager Suite
├── Kluis Control Center (GUI 관리 도구)
├── kluis-cli (명령행 도구)  
├── Kluis Agent (VM 내부 에이전트)
├── Kluis Runtime (실행 환경)
├── Kluis GPU Scheduler (GPU 스케줄러)
└── Kluis Templates (VM 템플릿)
```

### VM 관리 방식
- **VM 명칭**: Kluis Instance
- **관리 단위**: 개별 VM들의 집합체
- **실행 모드**: 심리스/윈도우 모드 선택 가능
- **GPU 할당**: 순환 스케줄링 방식

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

### 5. 🎮 GPU 리소스 공유 시스템

#### 작동 방식
- **순환 스케줄링**: VM들이 순차적으로 GPU 전체를 독점 사용
- **전체 점유**: 각 VM이 GPU 리소스를 100% 사용하여 최대 성능 확보
- **작업 완료 기반 전환**: 한 VM이 GPU 작업을 완료하면 다음 VM으로 자동 전환

#### 핵심 특징
- **사용자 정의 순서**: GPU 사용 순서를 직접 지정 가능
- **횟수 기반 제어**: 같은 VM을 여러 번 배치하여 더 많은 GPU 작업 기회 할당
- **선택적 참여**: GPU 큐에서 제외된 VM은 GPU 접근 불가
- **실시간 조정**: 운영 중에도 GPU 스케줄 동적 변경 가능

#### GPU 스케줄링 예시
```
GPU 큐: [VM-A, VM-B, VM-A, VM-C, VM-A]
처리 순서:
1. VM-A 작업 완료 → 2. VM-B 작업 완료 → 3. VM-A 작업 완료 → 
4. VM-C 작업 완료 → 5. VM-A 작업 완료 → 다시 1번부터 반복
결과: VM-A가 5번 중 3번, VM-B가 1번, VM-C가 1번 GPU 작업 수행
```

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

### GPU 리소스 관리
```bash
# GPU 스케줄 관리
kluis gpu queue                    # 현재 GPU 큐 상태 확인
kluis gpu queue set <vm1> <vm2> <vm3>  # GPU 큐 순서 설정
kluis gpu queue add <vm>           # VM을 GPU 큐에 추가
kluis gpu queue remove <vm>        # VM을 GPU 큐에서 제거
kluis gpu queue clear              # GPU 큐 전체 초기화
kluis gpu queue repeat <vm> <count> # VM을 지정 횟수만큼 큐에 추가

# GPU 사용 현황
kluis gpu status                   # GPU 사용 상태 및 현재 소유 VM
kluis gpu monitor                  # 실시간 GPU 작업 완료 모니터링
kluis gpu history                  # GPU 작업 기록 확인
kluis gpu next                     # 다음 차례 VM 확인

# GPU 정책 설정
kluis gpu policy set-mode <mode>   # 스케줄링 모드 (queue/priority/auto)
kluis gpu policy auto-queue        # 작업 부하 기반 자동 큐 조정
kluis gpu policy wait-timeout <sec> # 작업 대기 시간 초과 설정

# 개별 VM GPU 설정
kluis gpu assign <vm>              # VM에 GPU 접근 권한 부여
kluis gpu revoke <vm>              # VM의 GPU 접근 권한 제거
kluis gpu force-next               # 현재 작업 강제 종료 후 다음 VM으로
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
- **GPU 현황**: 현재 GPU 소유 VM 및 큐 진행 상태
- **빠른 액션**: VM 생성, 백업, 스냅샷

### VM 관리 패널
- **VM 카드 뷰**: 각 VM을 카드로 표시
- **실행 제어**: 시작/중지/재시작 버튼
- **모드 선택**: 심리스/윈도우 모드 토글
- **GPU 상태**: VM별 GPU 접근 권한 및 우선순위 표시
- **설정 관리**: 리소스, 네트워크, 공유 설정

### 🎮 GPU 리소스 센터
- **GPU 큐 관리**: 드래그 앤 드롭으로 순서 조정
- **실시간 모니터링**: GPU 작업 상태 및 현재 소유 VM 표시
- **작업 횟수 설정**: VM별 연속 GPU 작업 횟수 조정
- **스케줄링 정책**: 큐 기반/우선순위/자동 모드 전환
- **작업 통계**: GPU 작업 완료 히스토리 및 처리량 분석

#### GPU 큐 관리 인터페이스
```
┌─────────────────────────────────────────────────────────────┐
│  GPU 스케줄링 큐                                            │
├─────────────────────────────────────────────────────────────┤
│  1. [VM-Gaming]     ████████████████████░░░░  3회 연속      │
│  2. [VM-Work]       ████░░░░░░░░░░░░░░░░░░░░  1회           │
│  3. [VM-Gaming]     (큐에서 3번째)                          │
│  4. [VM-Gaming]     (큐에서 4번째)                          │
│                                                             │
│  [드래그하여 순서 변경] [+ VM 추가] [- VM 제거]             │
│                                                             │
│  현재 GPU 소유: VM-Gaming (2번째 작업 진행 중)              │
│  다음 차례: VM-Work                                         │
│  대기 중인 작업: 3개                                        │
└─────────────────────────────────────────────────────────────┘
```

### 애플리케이션 센터
- **통합 런처**: 모든 VM 앱을 통합 실행
- **앱 스토어**: VM별 애플리케이션 설치
- **바로가기 관리**: 데스크톱 통합 설정
- **GPU 요구사항**: 앱별 GPU 필요 여부 표시

### 백업 및 스냅샷
- **스냅샷 관리**: 시각적 타임라인 뷰
- **자동 백업**: 스케줄 설정 및 정책 관리
- **복원 도구**: 원클릭 롤백 기능

## GPU 스케줄링 시스템 상세

### 스케줄링 알고리즘
- **큐 기반**: 큐의 각 VM을 순차적으로 처리 (작업 완료 시 다음으로)
- **작업 단위**: GPU 작업 완료를 기준으로 다음 VM으로 전환
- **부하 감지**: GPU 작업 대기열에 따른 동적 우선순위 조정

### 전환 메커니즘
- **작업 완료 감지**: GPU 작업 종료 신호 실시간 모니터링
- **컨텍스트 전환**: GPU 상태 저장 후 다음 VM에 GPU 소유권 이전
- **대기열 관리**: 각 VM의 GPU 작업 요청을 큐에서 순차 처리

### 성능 최적화
- **즉시 전환**: 작업 완료 즉시 다음 VM으로 전환 (지연 없음)
- **상태 보존**: VM별 GPU 메모리 컨텍스트 독립 유지
- **효율적 큐잉**: 작업 대기 중인 VM만 큐에 포함하여 리소스 낭비 방지

## 설치 및 구성

### Kluis Manager 설치
```bash
# Kluis OS에서 (기본 포함)
sudo pacman -S kluis-manager kluis-gpu-scheduler

# 다른 Linux 배포판에서
wget kluis-manager-installer.sh
sudo ./kluis-manager-installer.sh --gpu-support
```

### GPU 지원 요구사항
- **GPU**: NVIDIA/AMD GPU (VFIO 지원)
- **드라이버**: 최신 그래픽 드라이버
- **IOMMU**: CPU 및 마더보드 IOMMU 지원
- **메모리**: 시스템 RAM 8GB 이상 권장

### 초기 설정
1. **서비스 시작**: `systemctl enable kluis-manager kluis-gpu-scheduler`
2. **GPU 감지**: 자동 GPU 하드웨어 검색 및 설정
3. **GUI 실행**: Kluis Control Center 시작
4. **첫 VM 생성**: 설정 마법사 실행
5. **GPU 테스트**: GPU 공유 기능 테스트 및 검증
6. **환경 구성**: 공유 설정 및 템플릿 준비

## 사용자 워크플로우

### 일반 사용
1. **앱 실행**: 데스크톱 아이콘 → 자동으로 적절한 VM에서 실행
2. **모드 변경**: 우클릭 → "새 창에서 열기" (독립 윈도우)
3. **VM 관리**: Control Center에서 상태 확인 및 제어
4. **GPU 모니터링**: 실시간 GPU 작업 진행 상황 및 큐 상태 확인

### GPU 리소스 관리
1. **GPU 큐 설정**: Control Center → GPU 센터 → 큐 순서 드래그 앤 드롭
2. **작업 횟수 조정**: VM별 연속 GPU 작업 횟수 설정
3. **작업 모니터링**: GPU 작업 완료 및 큐 진행 상황 실시간 확인
4. **정책 변경**: 스케줄링 모드 및 대기 시간 초과 설정

### VM 생성 및 관리
1. **새 VM**: Control Center → "새 VM 생성" → 템플릿 선택
2. **GPU 설정**: VM 생성 시 GPU 접근 권한 설정
3. **초기 설정**: 비심리스 모드에서 OS 설치 및 구성
4. **GPU 테스트**: GPU 기능 동작 확인
5. **운영 모드**: 설정 완료 후 심리스 모드로 전환

---

# 🔄 통합 운영 방식

## Kluis OS + Kluis Manager
- **기본 제공**: Kluis OS에는 Kluis Manager가 기본 설치
- **GPU 최적화**: OS 레벨에서 GPU 가상화 최적화 제공
- **독립 설치**: 다른 Linux 배포판에도 Kluis Manager 설치 가능
- **완전 통합**: OS와 매니저가 함께 최적의 사용자 경험 제공

## GPU 공유의 이점
- **효율성**: GPU 유휴 시간 최소화로 전체 작업 처리량 향상
- **비용 절감**: 하나의 GPU로 여러 VM이 고성능 작업 순차 수행
- **유연성**: 사용자 요구에 따른 동적 작업 순서 재할당
- **격리성**: VM 간 GPU 작업 완전 격리로 보안성 확보
- **공정성**: 큐 기반으로 모든 VM이 공평한 GPU 접근 기회 보장

## 확장성
- **플러그인**: 추가 기능을 위한 확장 모듈
- **API**: 다른 도구와의 통합을 위한 REST API
- **템플릿**: 커뮤니티 기반 VM 템플릿 공유
- **GPU 확장**: 다중 GPU 시스템 지원

## 호환성
- **OS 독립성**: Kluis Manager는 다양한 Linux에서 동작
- **VM 호환성**: 표준 QEMU/KVM 기반으로 높은 호환성
- **GPU 지원**: NVIDIA, AMD, Intel GPU 모두 지원
- **데이터 이식성**: VM 이미지 및 설정의 자유로운 이동

## 성능 벤치마크
- **작업 전환 시간**: 평균 즉시 전환 (작업 완료 감지 후 10ms 이내)
- **메모리 오버헤드**: VM당 추가 VRAM 사용량 64MB 이하
- **CPU 부하**: GPU 스케줄러 CPU 사용률 0.5% 이하
- **처리량**: 단일 GPU 대비 95% 이상의 총 작업 처리 성능 유지
- **큐 효율성**: 대기 중인 VM만 스케줄링하여 유휴 시간 최소화
