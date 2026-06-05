**Version: s26.1.6**

*Latest updates made on 6/5/2026*
# AER Embedded Systems Documentation
Welcome to the software and systems doc page for Anteater Electric Racing.
This site is the shared starting point for onboarding, architecture, and implementation details across the team.


## UPDATE for Spring Quarter 2026:

[[KZ to MZ changes]](./firmware/mzchanges.md)

Currently pushing to make FSAE competition in June 2026.

### Start Here:
- New to the project? Begin with onboarding:
	- [Data Acquisiton Onboarding](./dac/onboarding.md)
	- [Firmware Onboarding](./firmware/onboarding.md)
- Looking for system-level context?
	- [System Architecture](./firmware/sysarchitecture.md)

## Subteam Split

#### Firmware

Firmware owns embedded control and safety-critical behavior on vehicle modules.
- CCM control firmware (sensor I/O, control logic, telemetry) [(*CCM Overview*)](./firmware/ccm/ccm.md)
- PCC firmware for precharge sequencing and HV/LV safety interactions [(*PCC Overview*)](./firmware/pcc/pcc.md)
- FreeRTOS task scheduling and CAN message flow [(*Tasks & Scheduling*)](./firmware/ccm/threads.md)
- Vehicle state machine, fault handling, power limiting, and wheel-speed sensing

#### Data Acquisition (DAC)
The DAC side focuses on telemetry transport, storage, visualization, and simulation tooling.
- CAN ISO-TP telemetry ingestion and decoding [(*CAN docs*)](./dac/can/can.md)
- MQTT broker startup and configuration [(*MQTT docs*)](./dac/mqtt/mqtt.md)
- Telemetry fan-out to MQTT and TDengine [(*Send docs*)](./dac/send/send.md)
- Dashboard + Raspberry Pi telemetry services
- Simulator efforts (Godot, inverter simulation, RL) [(*Simulator docs*)](./dac/simulator/simulator.md)


## Current Focus Areas

- Tuning new Inverter and Motor Setup
- Thermal tuning to allow for maximum motor utilization
- Launch control tuning and slip-ratio based torque shaping
- Focus on tech compliance as per FSAE rules

## Contributing to Docs
This site is built with `mdBook` and Mermaid support.
- Add or update markdown pages under `src/`
- Keep navigation in sync through `src/SUMMARY.md`
- Run locally with `mdbook serve`

If you are unsure where new content belongs, start in the relevant onboarding page and link deeper implementation details from there.
