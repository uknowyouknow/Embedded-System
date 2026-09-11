# 1주차: 임베디드 시스템 개요 및 Raspberry Pi 4 부팅/초기 설정

## 1. 임베디드 시스템과 범용 컴퓨터의 비교

### 개념 정리
- **임베디드 시스템(Embedded System)**: 더 큰 제품 내부에서 **특정 기능**을 수행하기 위해 프로세서, 메모리, 입출력 장치가 결합된 전용 컴퓨터 시스템.
- **범용 컴퓨터 vs 전용 컴퓨터**:
  - **범용 컴퓨터 (PC, Laptop)**: 다양한 소프트웨어 설치 가능, 풍부한 자원(CPU, RAM), 사용자 편의성 중심.
  - **전용 컴퓨터 (Embedded/ECU)**: 단일/특정 목적 수행, 제한된 시스템 자원, **실시간성(Real-time)** 및 **고신뢰성/안전성** 필수.
  - **Raspberry Pi (SBC)**: Linux OS 기반의 범용성을 유지하면서 40-pin GPIO를 통해 센서/액추에이터 제어가 가능하여 범용성과 전용성을 연결해주는 역할을 수행.

---

## 2. Real-time Requirements & Edge AI

### 실시간성 (Real-time Properties)
- **Fast vs Real-time**: 빠른 처리속도보다 **정해진 마감 시간(Deadline) 내에 응답**하는 것이 핵심.
- **Hard Real-time**: 마감 시간을 어길 경우 시스템 전체에 치명적 실패 발생 (예: 자동차 에어백, 심박조율기).
- **Soft Real-time**: 일부 지연이 허용되나 서비스 품질이 저하됨 (예: 비디오 스트리밍).

### Edge AI vs Cloud AI
| 구분 | Edge AI (디바이스 내 AI) | Cloud AI |
| :--- | :--- | :--- |
| **응답 지연** | 낮음 (즉시 로컬 제어) | 높음 (네트워크 왕복시간 발생) |
| **네트워크 의존성** | 낮음 (오프라인 동작 가능) | 높음 (인터넷 필수) |
| **개인정보 보호** | 높음 (데이터 로컬 처리) | 낮음 (클라우드로 데이터 전송) |
| **계산 성능** | 제한적 (경량 모델 TinyML 활용) | 매우 높음 (대규모 GPU 연산) |

---

## 3. Raspberry Pi 4 하드웨어 및 부팅 매커니즘

### 주요 스펙 및 인터페이스
- **SoC**: Broadcom BCM2711 (Quad-core ARM Cortex-A72 @ 1.5GHz)
- **RAM**: LPDDR4 (2GB / 4GB / 8GB)
- **Storage**: microSD 카드 (Boot 파티션 FAT32 + Root 파일시스템)
- **I/O**: 40-pin GPIO (3.3V 신호 레벨 - **5V 직접 인가 금지**), Dual micro-HDMI (최대 4K), USB 3.0/2.0, Gigabit Ethernet, Wi-Fi/Bluetooth.

### 부팅 6단계 (Booting Sequence)
1. **전원 공급**: USB-C (5V / 3A) 안정적 전원 인가
2. **EEPROM 부트로더**: 저장장치 탐색
3. **microSD Boot 파티션**: `/boot`에서 커널 및 초기 부팅 파일(`config.txt`, `start4.elf`) 읽기
4. **Linux 커널 로드**: 하드웨어 및 장치 드라이버 초기화
5. **systemd 서비스**: 네트워크, SSH 및 백그라운드 서비스 시작
6. **로그인 / Desktop / CLI**: 사용자 작업 환경 도달

---

## 4. 안전 가이드라인 (Circuit & Power Safety)

1. **GPIO 신호 전압 레벨**: **3.3V 전용**. 5V 전압을 직접 입력할 경우 SoC 손상.
2. **LED 제어**: 반드시 **220Ω ~ 330Ω 전류 제한 저항**을 직렬 연결.
3. **고전류 장치**: 모터, 릴레이 등은 GPIO에 직접 연결하지 않고 **드라이버 모듈** 사용.
4. **배선 변경**: 반드시 **전원이 차단된 상태(OFF)**에서 회로 배선 작업 수행.
5. **안전한 종료 (Safe Shutdown)**:
   - 파일시스템(microSD) 손상 및 데이터 저널링 오류를 방지하기 위해 **강제 전원 차단 금지**.
   - 반드시 명령어로 안전하게 종료:
     ```bash
     sudo shutdown -h now
     ```
   - 초록색 ACT LED의 점멸이 완전히 멈춘 후 USB-C 전원 케이블 분리.

---

## 5. 터미널 주요 시스템 확인 명령어

```bash
# 1. 호스트 이름 및 IP 주소 확인
hostname
hostname -I

# 2. OS 및 커널 정보 확인
cat /etc/os-release
uname -a

# 3. CPU, 메모리, 저장공간 상태 확인
lscpu
free -h
df -h

# 4. 시간 및 Python 버전 확인
timedatectl status
python3 --version
