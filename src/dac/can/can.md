Author: Mikayla Lee
# AER Telemetry over CAN ISO-TP

## Overview

This system receives motor-controller telemetry over CAN ISO-TP, decodes it into strongly typed data, and forwards it for downstream processing. It also includes vCAN-based test utilities for CI.

---

## Function: `read_can_hardware`

**Purpose:** Continuously reads and decodes telemetry packets from `can0`

**Workflow:**

- Opens an ISO-TP socket on `can0` (source ID `0x666`, destination ID `0x777`)
- Retries socket creation on failure with a 1-second backoff
- Reads incoming packets asynchronously, validates packet length, and parses raw bytes into `TelemetryData` using Deku
- Captures a timestamp with `now_ms()` and forwards via `send_message(data, ts)`

**Error Handling:**

- Logs socket creation failures and malformed packets
- Logs packets with trailing bytes after decode
- Re-enters socket setup if packet reads stop or fail

---

## Function: `read_can`

Dispatches to hardware ISO-TP reception (`can0`) or synthetic mode when the `synthetic` feature flag is enabled.

---

## Function: `read_can_synthetic`

Publishes default `TelemetryData` at a ~1 ms interval for testing and CI. Logs throughput approximately once per second.

---

## Struct: `TelemetryData`

Represents a fully decoded MCU telemetry packet received over CAN ISO-TP. Fields are decoded little-endian from a fixed-layout binary frame.

| Field | Type | Notes |
|---|---|---|
| `apps_travel` | `f32` | Normalized accelerator pedal travel |
| `bse_front` | `f32` | Front brake sensor |
| `bse_rear` | `f32` | Rear brake sensor |
| `imd_resistance` | `f32` | Isolation monitoring measured resistance |
| `imd_status` | `u32` | Raw IMD status bits — interpret against MCU spec |
| `pack_voltage` | `f32` | HV pack voltage |
| `pack_current` | `f32` | HV pack current |
| `soc` | `f32` | Battery state of charge |
| `discharge_limit` | `f32` | Current discharge limit |
| `charge_limit` | `f32` | Current charge limit |
| `low_cell_volt` | `f32` | Minimum cell voltage |
| `high_cell_volt` | `f32` | Maximum cell voltage |
| `avg_cell_volt` | `f32` | Average cell voltage |
| `motor_speed` | `f32` | Motor rotational speed |
| `motor_torque` | `f32` | Instantaneous output torque |
| `max_motor_torque` | `f32` | Configured torque limit |
| `motor_direction` | `MotorRotateDirection` | |
| `motor_state` | `MotorState` | |
| `mcu_main_state` | `MCUMainState` | |
| `mcu_work_mode` | `MCUWorkMode` | |
| `mcu_voltage` | `f32` | MCU DC bus voltage |
| `mcu_phase_current` | `f32` | MCU phase current |
| `mcu_current` | `f32` | MCU current draw |
| `motor_temp` | `i32` | Motor temperature |
| `mcu_temp` | `i32` | MCU temperature |
| `mcu_warning_level` | `MCUWarningLevel` | Aggregated MCU warning severity |
| `shocktravel1`–`4` | `f32` | Shock travel channels 1–4 |
| `dc_main_wire_over_volt_fault` | `bool` | |
| `motor_phase_curr_fault` | `bool` | |
| `mcu_over_hot_fault` | `bool` | |
| `resolver_fault` | `bool` | |
| `phase_curr_sensor_fault` | `bool` | |
| `motor_over_spd_fault` | `bool` | |
| `drv_motor_over_hot_fault` | `bool` | |
| `dc_main_wire_over_curr_fault` | `bool` | |
| `drv_motor_over_cool_fault` | `bool` | |
| `dc_low_volt_warning` | `bool` | |
| `mcu_12v_low_volt_warning` | `bool` | |
| `motor_stall_fault` | `bool` | |
| `motor_open_phase_fault` | `bool` | |
| `over_current` | `bool` (1 bit) | System-level protection flag |
| `under_voltage` | `bool` (1 bit) | Supply below valid range |
| `over_temperature` | `bool` (1 bit) | Temperature exceeded safe limit |
| `apps` | `bool` (1 bit) | APPS fault flag |
| `bse` | `bool` (1 bit) | BSE fault flag |
| `bpps` | `bool` (1 bit) | BPPS fault flag |
| `apps_brake_plaus` | `bool` (1 bit) | Accelerator/brake plausibility violation |
| `low_battery_voltage` | `bool` (1 bit) | Battery low warning; followed by 24-bit pad |