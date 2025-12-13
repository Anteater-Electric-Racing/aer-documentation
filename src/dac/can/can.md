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
* Validates packet length (**expected: 58 bytes**)
* Decodes raw bytes into typed telemetry fields

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

* Serializes `TelemetryData` using `bincode`
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

* Driver input: accelerator pedal travel (`apps_travel`)
* Motor metrics: speed, torque, max torque, direction, state
* MCU status: main state, work mode, voltage, current
* Thermal data: motor and MCU temperatures
* Fault flags: over-voltage, over-current, stall, phase current, overspeed
* Warning level: MCU warning severity
* Debug channels: `debug_0` – `debug_3`

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

Each enum includes a `from_byte()` constructor for safe decoding from raw CAN payloads.

---

## CAN Reader Summary

### Function: `read_can`

**Purpose:** Continuously reads and decodes telemetry packets from `can0`

**Workflow:**

* Attempts to open an ISO-TP socket (retries on failure)
* Reads incoming packets asynchronously
* Validates packet length
* Parses raw bytes into `TelemetryData`
* Forwards decoded telemetry via `send_message()`

**Error Handling:**

* Logs socket creation failures
* Logs malformed packet lengths
* Continues operation without crashing

---

## Test: ISO-TP Telemetry Transmission

### Test: `test_send_telemetry_over_isotp`

**Purpose:** Verifies ISO-TP telemetry transmission over a virtual CAN interface

### Steps

#### 1. Prepare Test Data

* Creates a `TelemetryData` instance with representative values

#### 2. Send Telemetry

* Serializes the struct using `bincode`
* Transmits the packet over ISO-TP on `vcan0`

#### 3. Validate Environment

* Uses vCAN setup provided by the GitHub Actions workflow (`test.yml`)
* Confirms ISO-TP send path functions correctly

---

## Key Points

* Uses `tokio_socketcan_isotp` for asynchronous CAN ISO-TP communication
* Clean separation between production (`can0`) and test (`vcan0`) interfaces
* Designed for reliability in embedded deployment and CI environments
* Documentation aligns directly with packet structure and test behavior
