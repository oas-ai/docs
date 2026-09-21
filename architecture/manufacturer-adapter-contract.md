# Manufacturer Adapter Contract

## 역할

Manufacturer Adapter는 DBC decoder의 OEM-specific `DecodedCanMessage`를 OAS Canonical `VehicleState`로 변환하는 유일한 계층이다.

```text
DecodedCanMessage → Manufacturer Adapter → VehicleState → Application
```

Adapter 구현은 `manufacturer/platform` 단위로 분리한다. 초기 Reference Vehicle은 `GENESIS_RG3`이지만, 실제 signal mapping은 검증된 DBC와 CAN log가 확보된 뒤에만 추가한다.

## 인터페이스 초안

```rust
trait ManufacturerAdapter {
    type Error;
    fn apply(&mut self, message: &DecodedCanMessage) -> Result<(), Self::Error>;
    fn vehicle_state(&self) -> VehicleState;
}
```

`apply`는 수신된 message만 처리한다. adapter는 CAN socket, DBC file I/O, network transport를 직접 소유하지 않는다. 이 분리는 unit test에서 decoded fixture를 직접 주입할 수 있게 한다.

## 상태 규칙

- `VehicleState`는 SI 단위를 사용한다.
- 관측되지 않은 값은 `None` 또는 Protocol Buffers `optional` 부재로 유지한다.
- OEM signal의 scale, offset, enum은 Adapter 내부에서 변환한다.
- Platform ID는 `MANUFACTURER_PLATFORM` 형식이며 연식·트림·ADAS 차이는 capability metadata로 처리한다.
- 상태 freshness, counter, checksum, value range의 검증 결과는 별도 validity 모델로 확장한다.

## Safety 경계

이 계약은 read-only 상태 변환만 정의한다. Vehicle Control은 이 interface에 추가하지 않으며 다음의 별도 경로를 사용한다.

```text
Application → VehicleControl → SafetyModel → Manufacturer Controller → CAN TX
```

Raw CAN TX는 Adapter 또는 일반 Application에 노출하지 않는다.

## 검증 전략

1. DBC parser unit test
2. Adapter unit test: decoded message fixture → expected `VehicleState`
3. 실제 CAN log regression test
4. Hardware-in-the-loop test

Vehicle Control 또는 Safety 영향을 주는 변경은 3, 4단계와 별도의 Safety review가 필요하다.
