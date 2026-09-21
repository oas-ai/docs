# DBC Decoding Boundary

## 목적

`can`은 CAN/CAN FD frame의 transport-level 표현을 제공하고, DBC decoder는 승인된 DBC를 사용해 frame을 물리량 signal로 해석한다. 이 단계는 차량의 의미를 판단하거나 Vehicle Control을 수행하지 않는다.

```text
CAN Frame → DBC selection → message decode → DecodedCanMessage → Manufacturer Adapter
```

## 입력과 출력

입력은 `can::CanFrame`, 관측 timestamp, bus 식별자와 현재 Platform metadata다. DBC selection은 `dbc` registry에서 provenance와 SHA-256 검증을 통과한 파생 DBC만 사용한다.

출력 `DecodedCanMessage`에는 DBC message 식별자·frame ID, timestamp·bus, DBC signal name, physical value와 unit 또는 enum text만 담는다. OEM signal name은 이 경계를 벗어나지 않으며 OAS public API가 아니다.

## Rust 경계 초안

```rust
trait FrameDecoder {
    type Error;
    fn decode(&self, frame: &CanFrame, context: DecodeContext)
        -> Result<Option<DecodedCanMessage>, Self::Error>;
}
```

`None`은 현재 DBC가 해당 frame을 정의하지 않은 정상 상황이다. 정의된 message의 DLC, multiplexing 또는 signal decode가 실패한 경우만 `Error`다. decoder는 송신 기능을 갖지 않는다.

## 오류와 품질

- 승인되지 않았거나 hash가 다른 DBC는 로드하지 않는다.
- 알려지지 않은 frame은 상태를 변경하지 않는다.
- 해석 실패와 timestamp 누락은 Adapter가 state validity를 판단할 수 있도록 전달한다.
- 실제 CAN log fixture로 DBC decode 결과를 regression test한다.

## 하지 않는 일

- CAN frame 송신 또는 제어 명령 생성
- OEM signal을 `VehicleState` field로 직접 노출
- 실제 차량·DBC 정보의 추측
