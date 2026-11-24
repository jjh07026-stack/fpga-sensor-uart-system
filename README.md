# Basys-3 Sensor & UART Control System

미니 프로젝트로 만든 Watch · Stopwatch와 UART · FIFO를 합치고 DHT11 온습도 센서와 HC-SR04 초음파 센서를 추가해,
버튼 · 스위치와 PC UART 명령으로 제어하는 통합 시스템입니다.

- **기간**: 2026.11 (대한상공회의소 AI 시스템반도체 설계 2기 팀 프로젝트)
- **인원**: 4명
- **툴/보드**: Verilog HDL, Vivado, XSim, Digilent Basys-3 (Artix-7 XC7A35T)

## 담당 역할

| 이름 | 담당 |
|---|---|
| **정준혁** | **전체 모듈 통합, Control Unit 설계, 전체 검증** |
| 김성은 | ASCII Decoder / Sender 구현 및 검증 |
| 이시원 | DHT11 센서 유닛 구현 및 검증 |
| 황영빈 | SR04 센서 유닛 구현 및 검증 |

## 주요 기능

- 4개 모드 전환: Stopwatch · Watch · SR04 거리 · DHT11 온습도 → FND 표시
- PC에서 ASCII 명령(`r`, `c`, `u`, `l`, `0`~`3`)으로 버튼 · 스위치와 같은 제어, 측정값을 PC로 송신
- 예외 처리: 센서 무응답(timeout)과 checksum 오류 시 `xxxx` 출력

## 결과 요약

| 항목 | 결과 |
|---|---|
| 통합 시나리오 시뮬레이션 (시계 · SR04 · DHT11 · 예외 처리) | **4 / 4 PASS** |
| 통합 중 발견한 문제 | **4 / 4 해결** |
| Basys-3 보드 동작 | 확인 |

## 통합 검증 중 발견하고 해결한 문제

| 현상 | 원인 | 해결 |
|---|---|---|
| UART로 모드 전환(`1`) 후 이전 모드 값이 송신됨 | 모드 전환과 같은 클록에 송신 요청 | 모드가 바뀐 다음 클록에 송신 |
| 10 cm인데 `0000cm` 송신 | `distance`가 `done` 다음 클록에 갱신 | 다음 클록에 송신 |
| `1`, `0` 연속 입력 시 응답이 1개만 옴 | Sender가 busy일 때 들어온 요청을 무시 | `sender_ready`까지 요청 유지 |
| DHT11 무응답 시 PC에 아무것도 송신되지 않음 | timeout 때 `done`이 나오지 않음 | timeout에도 `done`, `valid = 0` → `xxxx` |

## 파일

```
rtl/          project_top.v (top), control_unit.v, control_btn_unit.v,
              stopwatch.v, watch.v, fnd_controller_new.v, btn_db.v,
              uart_fifo.v, uart_rx.v, uart_tx.v, fifo.v, top_uart_fifo_ascii.v,
              ascii_decoder.v, ascii_sender.v, binary_to_ascii.v,
              dht11_module.v, sr04.v, sr04_controller.v, sr04_datapath.v, us_tick_gen.v
tb/           tb_project_top.v (통합 시나리오), tb_system_exception.v (예외 처리), tb_trouble.v (문제 재현)
constraints/  basys3.xdc
scripts/      wave_setup.tcl, wave_dht.tcl, wave_trouble.tcl (XSim 파형 설정)
docs/         report.pdf (결과 보고서)
```
