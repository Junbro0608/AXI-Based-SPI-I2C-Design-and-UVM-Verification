# SoC Design & Verification: AXI4-Lite based SPI & I2C Peripherals

이 프로젝트는 **AXI4-Lite 인터페이스**를 기반으로 한 **SPI 및 I2C 통신 모듈의 SoC 설계 및 UVM(Universal Verification Methodology) 검증** 과정을 담고 있습니다. FPGA(Basys3) 환경에서 마이크로프로세서(MicroBlaze)와 주변 장치 간의 효율적인 데이터 송수신 제어를 목표로 합니다.


## 📌 프로젝트 개요
- **목표**: 
  - AXI4-Lite 프로토콜을 준수하는 SPI/I2C Master 모듈 설계.
  - 설계된 모듈에 대한 UVM 기반 검증 환경 구축 및 신뢰성 확보.
  - Basys3 FPGA 보드를 활용한 실제 하드웨어 데이터 송수신 구현.
- **주요 기술 스택**: Verilog HDL, AXI4-Lite, UVM, FPGA Synthesis.

## 🛠 사용 툴 및 개발 환경
| 구분 | 내용 |
| --- | --- |
| **Language** | Verilog, SystemVerilog (UVM 1.2) |
| **SoC / FPGA Tool** | Xilinx Vivado, Vitis IDE |
| **Verification Tool** | Synopsys VCS, Verdi |
| **Hardware** | Basys3 (Artix-7 FPGA) |
| **Communication** | AXI4-Lite, SPI, I2C |

## 🏗 시스템 아키텍처 및 설계 내역
### 1. AXI4-Lite Interface
- 5개의 독립적인 채널(AW, W, B, AR, R)을 통한 양방향 통신 구현.
- MicroBlaze와 Peripheral(SPI/I2C) 간의 Register Read/Write 제어.

### 2. SPI & I2C Master Module
- **SPI**: Full Duplex 통신, 모드 설정(Clock Polarity/Phase), 시퀀셜 데이터 처리.
- **I2C**: Half Duplex 통신, Start/Stop Condition 제어, Slave 주소 지정 및 ACK 체크.

## 🧪 검증 및 테스트 (UVM Verification)
설계된 AXI Peripheral 모듈의 기능적 무결성을 확인하기 위해 UVM 환경을 구축했습니다.
- **Driver**: AXI Write/Read 트랜잭션 발생 및 SPI/I2C 제어 시퀀스 실행.
- **Monitor**: AXI 인터페이스와 Serial 인터페이스의 유효 데이터 캡처.
- **Scoreboard**: `axi_wdata`와 `slv_rx_data`, `axi_rdata`와 `slv_tx_data`를 비교하여 데이터 무결성 검증.
- **Result**: Random Test Case를 통한 시뮬레이션 및 Verdi를 이용한 파형 분석 수행.

## 🚀 하드웨어 합성 및 결과
- **Basys3 FPGA 합성**: Vivado를 이용한 비트스트림 생성 및 보드 다운로드.
- **동작 확인**: 두 대의 Basys3 보드를 활용하여 실제 SPI/I2C 통신 프로토콜 준수 여부 및 데이터 송수신 확인.

## 🔍 Trouble Shooting

### 1. SW DONE Check 로직 설계 (SoC 통합 과정)
* **Issue**: MicroBlaze(SW)가 하드웨어의 동작 완료 시점(`done`)을 인지하지 못해 다음 명령(`cmd`)을 제때 전달하지 못하는 문제 발생.
* **Cause**: 하드웨어 내부의 `done` 신호가 단 **1-tick(1 clock)** 동안만 발생하여, 상대적으로 느린 소프트웨어가 폴링(Polling) 방식으로 해당 신호를 포착하기에 시간이 너무 짧음.
* **Solution**: 하드웨어 설계 시 `done` 신호를 유지해 주는 **Done Flag Logic**을 추가하여, 소프트웨어가 상태 레지스터를 읽어 확인하기 전까지 신호를 유지하도록 개선.

### 2. SPI 통신 타이밍 미스매치 및 응답 속도 개선
* **Issue**: SPI Master-Slave 통신 시 데이터가 깨지거나 특정 주파수에서 통신이 불가능한 현상 발생.
* **Cause**: 
    1. 2개의 Flip-Flop을 거치는 기존 Edge 검출 방식이 응답 시간을 지연시킴.
    2. Slave 모듈의 데이터 출력(TX_data)이 순차 논리(Sequential Logic)로 설계되어 타이밍 마진을 확보하지 못함.
* **Solution**:
    1. Edge 검출 로직을 **1개의 Flip-Flop** 구조로 변경하여 신호 인지 속도 향상.
    2. 데이터 출력부를 **조합 논리(Combinational Logic)** 기반으로 재설계하여 전파 지연(Propagation Delay)을 최소화하고 안정적인 타이밍 확보.
---
**발표자**: 이준형
