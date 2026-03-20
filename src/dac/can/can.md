Author: Mikayla Lee
# CAN
# AER Telemetry over CAN ISO-TP

## CAN ISO-TP Telemetry Receiver & Test Pipeline

### Overview

This system receives telemetry data from the motor controller over CAN using ISO-TP, decodes the packet into strongly typed data, and forwards it for downstream processing. It also includes test utilities for transmitting and validating telemetry over a virtual CAN interface (vCAN) in CI.

---

## Workflow

### CAN ISO-TP Data Reception

* Listens on the physical CAN interface (`can0`) using ISO-TP sockets
* Receives multi-frame telemetry packets from the motor controller
* Validates packet length using `Deku`.
* Decodes raw bytes into typed telemetry fields using `Deku`.

### Telemetry Parsing

* Converts byte slices into numeric values (`f32`, `i32`) using little-endian decoding
* Maps state and mode bytes into enums representing motor and MCU state machines
* Parses fault flags into boolean values
* Assembles all decoded values into a `TelemetryData` structure

### Telemetry Forwarding

* Wraps decoded values in a `TelemetryData` struct
* Publishes telemetry via `send_message()`
* Uses the **"telemetry"** topic for downstream consumers

### ISO-TP Test Transmission (vCAN)

* Serializes `TelemetryData` using `Deku`
* Sends telemetry as a single ISO-TP packet over `vcan0`
* Used exclusively for testing and CI validation

---

## Key Features

* **Strongly Typed Telemetry** – Enums and structs ensure safe decoding of MCU and motor states
* **Resilient Socket Handling** – Automatically retries socket creation on failure
* **Clear Packet Contract** – Enforces a strict 58-byte telemetry frame layout
* **CI-Friendly Testing** – vCAN-based tests run in GitHub Actions
* **Extensible Design** – New telemetry fields and enums can be added with minimal refactoring

---

## Telemetry Data Summary

### Struct: `TelemetryData`

**Purpose:** Represents a fully decoded MCU telemetry packet received over CAN ISO-TP

**Contents:**

* `apps_travel` (`f32`) – Accelerator pedal travel as a normalized throttle input value.

* **Motor metrics**
    * `motor_speed` (`f32`) – Current motor rotational speed.
    * `motor_torque` (`f32`) – Instantaneous motor output torque.
    * `max_motor_torque` (`f32`) – Maximum available or configured motor torque limit.
    * `motor_direction` (`MotorRotateDirection`) – Commanded/observed motor rotation direction.
    * `motor_state` (`MotorState`) – Current high-level motor state (for example idle, driving, or fault).

* **MCU status**
    * `mcu_main_state` (`MCUMainState`) – Main inverter/controller state machine state.
    * `mcu_work_mode` (`MCUWorkMode`) – Active control mode (such as torque or speed mode).
    * `mcu_voltage` (`f32`) – Measured MCU DC bus/input voltage.
    * `mcu_current` (`f32`) – Measured MCU current draw.
    * `mcu_warning_level` (`MCUWarningLevel`) – Aggregated warning severity reported by the MCU.

* **Thermal data**
    * `motor_temp` (`i32`) – Motor temperature reading.
    * `mcu_temp` (`i32`) – MCU/controller temperature reading.

* **Fault flags (motor/controller)**
    * `dc_main_wire_over_volt_fault` (`bool`) – True when DC main wire voltage exceeds safe threshold.
    * `dc_main_wire_over_curr_fault` (`bool`) – True when DC main wire current exceeds safe threshold.
    * `motor_over_spd_fault` (`bool`) – True when motor speed is above allowed limit.
    * `motor_phase_curr_fault` (`bool`) – True when motor phase current fault is detected.
    * `motor_stall_fault` (`bool`) – True when motor stall condition is detected.

* **Safety/plausibility flags (bitfield bools)**
    * `over_current` (`bool`) – System-level over-current protection flag.
    * `under_voltage` (`bool`) – Supply voltage below valid operating range.
    * `over_temperature` (`bool`) – Temperature exceeded configured safe limit.
    * `apps` (`bool`) – Accelerator Pedal Position Sensor fault flag.
    * `bse` (`bool`) – Brake System Encoder/Sensor fault flag.
    * `bpps` (`bool`) – Brake Pedal Position Sensor fault flag.
    * `apps_brake_plaus` (`bool`) – Accelerator/brake plausibility violation flag.
    * `low_battery_voltage` (`bool`) – Battery voltage low warning/fault flag.

---

## Enum Definitions

### MotorState

* Off
* Precharging
* Idle
* Driving
* Fault

### MotorRotateDirection

* Standby
* Forward
* Backward
* Error

### MCUMainState

* Standby
* Precharge
* PowerReady
* Run
* PowerOff

### MCUWorkMode

* Standby
* Torque
* Speed

### MCUWarningLevel

* None
* Low
* Medium
* High

---

## CAN Reader Summary

### Function: `read_can_hardware`

**Purpose:** Continuously reads and decodes telemetry packets from `can0`

**Workflow:**

* Attempts to open an ISO-TP socket (retries on failure)
* Reads incoming packets asynchronously
* Validates packet length
* Parses raw bytes into `TelemetryData` using `Deku`.
* Forwards decoded telemetry via `send_message()`

**Error Handling:**

* Logs socket creation failures
* Logs malformed packet lengths
* Continues operation without crashing

---

## Key Points

* Uses `tokio_socketcan_isotp` for asynchronous CAN ISO-TP communication
* Clean separation between production (`can0`) and test (`vcan0`) interfaces
* Designed for reliability in embedded deployment and CI environments
* Documentation aligns directly with packet structure and test behavior
