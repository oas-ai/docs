# Palisade 2020 read-only product slice

## Data ownership

Approved opendbc provenance in `dbc` → subset decoder in `can` → SI adapter in `car` → framed `VehicleState` from `gateway` → Runtime freshness and playback policy → framed `HmiState` → Qt HMI.

Runtime owns capability decisions. The C++ bridge can invalidate expired transport data but cannot grant capability. DBC `raw_signals` remain diagnostic values, not a vehicle-health assessment or safety/control inputs. Synthetic DBC fixtures are the present development evidence; no real-car validation is claimed.

## Current product boundary

The deliverable is a Linux userspace read-only vehicle platform and a macOS Qt preview. It is not a bootable OS distribution, a certified cluster, an installed media player or a navigation system. These require target hardware, BSP/display integration and separately scoped services. There is no product CAN transmit API.

## Verification

- SDK: Buf formatting/lint and PR-base breaking checks; Rust generated-code tests.
- DBC: provenance manifest schema, SHA-256, registered-file coverage, confined paths and cantools parsing.
- CAN/car: synthetic DBC decoder and adapter regression tests.
- Gateway: Linux vCAN pipelines, restart tests, browser Viewer tests and actual Qt FIFO capture.
- Qt: C++ FIFO expiry/recovery tests and rendered demo states, with QML binding failures treated as errors.

Fresh state is not proof that every signal is valid or that the vehicle is healthy. Range and fuel remain absent. Demo states are explicitly labeled synthetic. The artwork is an original concept illustration, not a sensed environment or an exact Palisade model.

Build and capture commands live in `ohayessOS/apps/hmi/README.md`; deployment templates are in `gateway/packaging/systemd`. CAN receive is a product feature; fixture transmission is confined to vcan0 tests.
