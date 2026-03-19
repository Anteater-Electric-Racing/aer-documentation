**Version: w26.1.1**

*Latest updates made on 3/20/2026*
# AER Embedded Systems Documentation
Welcome to the software and systems doc page for Anteater Electric Racing.
This site is the shared starting point for onboarding, architecture, and implementation details across the team.
### Start Here:
- New to the project? Begin with onboarding:
	- [Data Acquisiton Onboarding](./dac/onboarding.md)
	- [Firmware Onboarding](./firmware/onboarding.md)
- Looking for system-level context?
	- [System Architecture](./firmware/sysarchitecture.md)

## Subteam Split
#### Data Acquisition (DAC)
The DAC side focuses on telemetry transport, storage, visualization, and simulation tooling.
- CAN ISO-TP telemetry ingestion and decoding [(*CAN docs*)](./dac/can/can.md)
- MQTT verification and publish/subscribe pipeline [(*MQTT docs*)](./dac/mqtt/mqtt.md)
- InfluxDB line protocol conversion and storage [(*InfluxDB docs*)](./dac/influxdb/influxdb.md)
- Dashboard + Raspberry Pi telemetry services
- Simulator efforts (Godot, inverter simulation, RL) [(*Simulator docs*)](./dac/simulator/simulator.md)

#### Firmware

Firmware owns embedded control and safety-critical behavior on vehicle modules.
- CCM control firmware (sensor I/O, control logic, telemetry) [(*CCM Overview*)](./firmware/ccm/ccm.md)
- PCC firmware for precharge sequencing and HV/LV safety interactions [(*PCC Overview*)](./firmware/pcc/pcc.md)
- FreeRTOS task scheduling and CAN message flow [(*Tasks & Scheduling*)](./firmware/ccm/threads.md)
- Vehicle state machine, fault handling, power limiting, and wheel-speed sensing

## Current Focus Areas

- Transitioning codebase to comp car with new Motor + Inverter
- CAN interrupt robustness and queue overflow validation
- Shock sensor integration and calibration for suspension telemetry
- Launch control tuning and slip-ratio based torque shaping
- Power limit behavior around rule thresholds and low-SOC derating
- Continued simulator expansion and PCC subsystem documentation

## Contributing to Docs
This site is built with `mdBook` and Mermaid support.
- Add or update markdown pages under `src/`
- Keep navigation in sync through `src/SUMMARY.md`
- Run locally with `mdbook serve`

If you are unsure where new content belongs, start in the relevant onboarding page and link deeper implementation details from there.
