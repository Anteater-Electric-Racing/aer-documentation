Author: Alistair Keiller
# Onboarding

## Relevant coding background to understand DAC

Git: [Video](https://www.youtube.com/watch?v=TFhbv6gw2Wo) || [Article](https://dev.to/ajmal_hasan/beginner-friendly-git-workflow-for-developers-2g3g)
Rust: [Book](https://doc.rust-lang.org/book/) && [Rustlings](https://github.com/rust-lang/rustlings/)
Cargo: [Guide (sections 1-2, optionally 3)](https://doc.rust-lang.org/cargo/getting-started/index.html)
Iced.rs: [Short Guide](https://book.iced.rs/index.html)
Tokio.rs: [Tutorial](https://tokio.rs/tokio/tutorial)

## Relevant hardware background to understand DAC

Raspberry Pi 5: [Fun Video](https://www.youtube.com/watch?v=UtLyX72-688)
Teensy 4.1: [Fun Video](https://www.youtube.com/watch?v=75IvTqRwNsE)
CAN protocol: [Video](https://www.youtube.com/watch?v=JZSCzRT9TTo)

## Codebase overview

Here is our codebase: [https://github.com/Anteater-Electric-Racing/embedded](https://github.com/Anteater-Electric-Racing/embedded)

### [fsae-dashboard](https://github.com/Anteater-Electric-Racing/embedded/tree/main/fsae-dashboard)

![dashboard](images/dashboard.png)

This dashboard displays current critical information to the driver.

### [fsae-raspi](https://github.com/Anteater-Electric-Racing/embedded/tree/main/fsae-raspi)

Runs on the Raspberry Pi and handles all telemetry ingestion and storage. It polls ISO-TP CAN data from `can0`, decodes it into typed telemetry, and fans it out to an embedded [rumqttd](https://github.com/bytebeamio/rumqtt) MQTT broker and a [TDengine](https://tdengine.com/) time-series database.

See the [CAN docs](./dac/can/can.md), [MQTT docs](./dac/mqtt/mqtt.md), and [Send docs](./dac/send/send.md) for details.