# OAS Documentation

OAS의 Architecture, Engineering Standard 및 공개 개발 문서를 관리합니다. 문서는 한국어로 작성하고, 식별자·API field·파일명은 영어를 사용합니다.

초기 상세 문서는 `engineering/` 아래에 추가합니다.

현재 Architecture 문서는 [architecture/](architecture/README.md)에서 관리합니다.

## 현재 상태

OAS는 AVN용 read-only 차량 상태 pipeline을 먼저 구현하고 있습니다. Genesis G80 2017의
opendbc 기반 DBC subset에서 속도·가속도·조향·제동·기어·휠 속도·SCC·저빔 상태를
`VehicleState` protobuf stream으로 전달합니다. 저빔은 HMI의 자동 라이트/다크 테마에
사용되며, Gateway 재시작 뒤에도 stream과 HMI가 복구되는 vCAN E2E를 갖추고 있습니다.

DBC 값의 의미가 아직 실차에서 확정되지 않은 도어·안전벨트·와이퍼·조명 값은
`raw_signals`에 원본 DBC 신호명과 physical value로 보존합니다. 이는 안전 판단이나 차량
제어에 사용하지 않습니다.

## 다음 작업

1. **DBC catalog 확장** — Genesis G80 DBC의 공조·도어·안전벨트·진단 신호를 raw catalog에
   추가하고 HMI Diagnostics에서 조회 가능하게 한다.
2. **실차 신호 승인** — Canable 수신 로그로 연식·시장·트림을 확인하고 raw 값의 enum 의미를
   검증한 뒤에만 `door.open`, `seatbelt.latched`, 공조 상태처럼 canonical field로 승격한다.
3. **Radxa 통합** — 실제 Linux 장비에서 SocketCAN, systemd 서비스, kiosk/WebView, 부팅·절전·재연결
   동작을 검증한다.
4. **AVN 기능 연결** — 승인된 read-only 상태를 HMI의 차량·공조·진단 화면에 표시한다. 차량 제어
   명령은 별도 Safety boundary와 위험 분석을 완료하기 전까지 추가하지 않는다.
5. **회귀 데이터** — 익명화한 실차 캡처를 최소 fixture로 정제해 DBC revision별 decoder·gateway·HMI
   E2E에 추가한다.

각 구성요소의 실행·배포 정보는 [gateway](../gateway/README.md),
[ohayessOS](../ohayessOS/README.md), [CAN](../can/README.md), [vehicle model](../car/README.md),
[SDK](../sdk/README.md) 문서를 따른다.
