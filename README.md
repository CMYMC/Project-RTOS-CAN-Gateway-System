# RTOS-CAN Ethernet Gateway System
**FreeRTOS 기반 CAN-Ethernet 데이터 중계 게이트웨이 프로젝트**

> **STM32 + FreeRTOS 환경에서 다수의 임베디드 노드가 생성하는 CAN 데이터를
중앙 게이트웨이에서 제어·통합하여 이더넷 기반 상위 시스템으로 전달하는 RTOS 기반 네트워크 프로젝트**

---

## 프로젝트 개요

RTOS-CAN-Ethernet Gateway System은 FreeRTOS와 LwIP를 활용하여
CAN으로 수신한 센서 데이터를 CAN ID 기준으로 분류하고,
UDP 전송 데이터 형식으로 구성해 Raspberry Pi로 전달하는 임베디드 게이트웨이 시스템입니다.

## 핵심 목표

- FreeRTOS 기반 멀티 태스크 구조 설계
- CAN 통신 + RTOS Queue 기반 데이터 전달 구조
- 임베디드 시스템에서 Ethernet/UDP 전송 구조 적용
- 수신과 전송 처리를 분리한 게이트웨이 구조 구현

---

## 구성도




<img width="382" height="377" alt="하드웨어" src="https://github.com/user-attachments/assets/3112a68f-4c6f-4afb-869a-c1a45ba9de4d" />

**시스템 아키텍처**

<img width="761" height="573" alt="image" src="https://github.com/user-attachments/assets/15ef9630-0f89-49f8-8daa-2f088613e7ea" />

- **Board A / B**

  - 센서 데이터 생성
  - CAN 메시지 송신
  
- **Board C (Gateway)**

  - CAN 메시지 수신 및 ID 기반 필터링
  - 메시지 유형에 따른 데이터 분기 처리
  - RTOS 기반 제어 흐름 관리
  - Ethernet을 통한 상위 시스템 연동

- **Raspberry Pi**
  - UDP 수신
  - 최종 데이터 출력

---

## 담당 역할
- CAN RX 인터럽트 기반 수신 처리 구현
- RTOS Message Queue를 활용한 데이터 전달 구조 구현
- CAN ID 기반 데이터 분류 및 UDP 전송 데이터 구성
- LwIP UDP 소켓을 활용한 Raspberry Pi IP/Port 송신 구현
---

## 수행 목표

- RTOS 환경에서 CAN 통신을 안정적으로 처리
- 멀티 태스크 환경에서 데이터 충돌 없이 통신
- 임베디드 시스템에 네트워크 계층 개념 적용
- 확장 가능한 Gateway 구조 설계

---

## 사용 기술

### Embedded / RTOS
![STM32](https://img.shields.io/badge/STM32-Nucleo--F429ZI-blue?style=plastic&logo)
![FreeRTOS](https://img.shields.io/badge/FreeRTOS-RTOS-green?style=plastic&logo)
![CMSIS](https://img.shields.io/badge/CMSIS-RTOS-lightgrey?style=plastic&logo)
![HAL](https://img.shields.io/badge/STM32-HAL-orange?style=plastic&logo)

### Communication
![CAN](https://img.shields.io/badge/CAN-Bus-red?style=plastic&logo)
![CAN Filter](https://img.shields.io/badge/CAN-Filter_Implemented-critical?style=plastic&logo)
![Frame](https://img.shields.io/badge/Data-Frame_Packing-yellow?style=plastic&logo)

### Toolchain
![CubeIDE](https://img.shields.io/badge/STM32-CubeIDE-blueviolet?style=plastic&logo)
![CubeMX](https://img.shields.io/badge/STM32-CubeMX_.ioc-informational?style=plastic&logo)

---

## 주요 구현 내용

1️⃣ FreeRTOS 데이터 처리 구조

- CAN RX 인터럽트에서 수신 데이터 확인
- 수신 데이터를 RTOS Message Queue에 저장
- Ethernet Task에서 Queue 데이터를 가져와 UDP 전송 데이터로 구성
- 수신 처리와 전송 처리를 분리해 메인 흐름의 부담을 줄임

👉 CAN RX ISR → RTOS Message Queue → Ethernet Task → UDP 송신 구조로 구현

2️⃣ CAN 통신 구현

- HAL CAN Driver 기반 송·수신
- CAN Filter 직접 설정
- 메시지 ID 기반 데이터 분기 처리
- 프레임 패킹을 통한 데이터 구조화
```c
typedef struct __attribute__((packed)) {
    uint8_t  start_byte;
    uint16_t can_id;
    uint8_t  dlc;
    uint8_t  data[8];
} UdpPacket;
```

UDP Header는 LwIP가 sendto() 호출 시 자동으로 구성하며,
위 구조체는 UDP Payload로 전송되는 사용자 데이터 영역입니다.

3️⃣ RTOS + CAN 데이터 전달 구조

- CAN RX Interrupt 발생 시 수신 데이터 확인
- 수신 데이터를 RTOS Message Queue에 저장
- Ethernet Task에서 Queue 데이터를 가져와 UDP 전송 데이터로 구성
- 수신 처리와 전송 처리를 분리해 구조를 단순화

👉 CAN RX ISR → RTOS Message Queue → Ethernet Task → UDP 송신

👉 RTOS 환경에서 실시간성 + 안정성 확보

4️⃣ 네트워크 개념 적용

- Board C를 Gateway 노드로 설정
- CAN → Ethernet 방향의 데이터 흐름 설계
- 스위치 및 게이트웨이의 데이터 처리·제어 구조를 임베디드 환경에 적용
- 중앙 제어 노드를 통한 데이터 필터링 및 분기 구조 설계

---

## 데이터 통합 및 DB 로그 결과

CAN 기반 분산 노드에서 수집된 데이터를 중앙 게이트웨이(Board C)에서
시간 기준으로 통합하고, Ethernet을 통해 상위 시스템(Raspberry Pi)으로 전달하여
DB에 스냅샷 형태로 저장한 결과입니다.

<img width="1149" height="600" alt="image" src="https://github.com/user-attachments/assets/4a31a15e-729a-4e88-9bee-45bcf1fcf332" />

- Board A(환경 센서)와 Board B(구동/거리 센서)의 데이터를 CAN으로 수신
- Gateway(Board C)에서 메시지 ID 기준 필터링 및 데이터 통합
- Ethernet(UDP) 기반으로 Raspberry Pi에 전달 후 SQLite DB에 저장

### DB 스냅샷 로그 예시

<img width="431" height="458" alt="image" src="https://github.com/user-attachments/assets/5935a76b-fc99-4860-b657-f8dda5a4eef5" />

- 1초 주기 스냅샷 방식으로 데이터 저장
- 센서/구동 데이터가 동일 타임스탬프로 정합성 있게 기록됨
- 분산 노드 데이터를 중앙에서 제어·관리하는 게이트웨이 구조 검증

---

## 트러블슈팅

### 1) Ethernet Task 생성 실패

- **문제**: Ethernet Task가 정상적으로 생성되지 않는 문제 발생
- **원인**: 제한된 MCU RAM 환경에서 Ethernet Task의 스택 크기를 과도하게 설정해 태스크 생성에 필요한 메모리를 확보하지 못함
- **해결**: Ethernet Task의 처리 범위와 사용 변수 범위를 고려해 스택 크기를 1024 Words로 조정
- **결과**: Ethernet Task가 정상적으로 생성되고 UDP 전송 루틴을 실행할 수 있음을 확인

### 2) FreeRTOS–HAL SysTick 자원 충돌로 인한 데드락
- **문제**: RTOS 구동 후 Ethernet(LwIP) 초기화 과정에서 시스템이 멈추며 태스크 전환이 발생하지 않음
- **원인**  
  - FreeRTOS와 STM32 HAL이 동일한 SysTick 타이머를 시간 기준으로 공유  
  - RTOS 스케줄링과 HAL/LwIP 초기화 로직이 동시에 SysTick을 사용하며 자원 충돌 발생  
  - 특정 모듈 문제가 아닌 RTOS–HAL 전반의 타이밍 자원 충돌
- **해결**  
  - HAL 타임베이스를 SysTick에서 하드웨어 타이머(TIM6) 기반으로 변경  
  - RTOS 스케줄러와 HAL 시간 관리 로직을 물리적으로 분리  
  - FreeRTOS tick과 HAL delay 함수 간 간섭 제거
- **결과**  
  - 데드락 문제 해결  
  - CAN 및 Ethernet Task 병행 실행 시에도 RTOS 정상 동작
    
### 3) CAN 수신 처리 구조 개선

- **개선 방향**: CAN 수신 처리를 Polling이 아닌 Interrupt 기반으로 구성
- **구현**: CAN RX 인터럽트에서 수신 데이터를 Queue에 넣고, Ethernet Task에서 후속 처리를 수행
- **의미**: 수신 처리와 전송 처리를 분리해 구조를 단순화하고 확장성을 높임
---

## 프로젝트를 진행하며 느낀점

- RTOS 기반 환경에서 통신 데이터 흐름을 설계한 경험
- CAN ID 기반 데이터 분류와 UDP 전송 데이터 구성에 대한 이해
- Queue 기반 데이터 전달 구조를 통한 수신·전송 처리 분리 경험
- LwIP 소켓을 활용한 IP/Port 기반 UDP 송신 구조 이해
