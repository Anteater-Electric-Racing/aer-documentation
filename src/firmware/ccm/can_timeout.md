Authors: Anushree Godbole, Kenneth Dao

# CAN Bus Fault Detection Implementation

## Overview

The CAN Bus fault detection subsystem is implemented to monitor whether CAN communication is running reliably. Its primary function is to detect when CAN messages stop being received within the required timing window and set a system fault when communication becomes stale.

This implementation monitors:

* CAN receive activity
* Latest healthy CAN message timestamp
* CAN communication timeout
* CAN fault state through the system fault bitmask

If no healthy CAN receive update occurs within the defined threshold, the system sets a CAN fault.

## System Architecture

The CAN fault detection system consists of:

1. CAN Receive Handler
2. CAN Health Timestamp (`canLatestHealthyStateTime`)
3. Current CAN Task Timestamp (`can_last_run_tick`)
4. FreeRTOS Tick-Based Timing Check
5. Bitmask-Based Fault System

Each time a CAN message is successfully received, the system updates the latest healthy CAN timestamp. The CAN monitoring logic then compares the current CAN runtime tick against the latest healthy receive time.

## Fault Detection Strategy

The system uses a time-based communication timeout approach.

The CAN age is calculated as:

```cpp
canAgeMs = (can_last_run_tick - canLatestHealthyStateTime) *
           portTICK_PERIOD_MS;
```

If the CAN age exceeds the fault threshold, the CAN fault is set:

```cpp
if (canAgeMs > CAN_FAULT_TIME_THRESHOLD_MS) { // 100 ms
    Faults_SetFault(FAULT_CAN);
} else {
    Faults_ClearFault(FAULT_CAN);
}
```

This means the CAN system is considered healthy only if a recent CAN receive event has occurred within the allowed time window.

## Fault Representation

CAN faults are represented using the vehicle fault bitmask.

| Fault     | Bit | Value       |
| --------- | --- | ----------- |
| CAN Fault | 7   | `0x1 << 7` |

The CAN fault is defined as a single bit in the shared fault mask:

```cpp
FAULT_CAN = 0x1 << 7
```

When the CAN fault condition is detected, this bit is set. When CAN communication returns to a healthy state, the bit is cleared.

## Timing Relationship

The CAN fault detection uses a **100 ms timeout threshold**:

* **Healthy CAN communication:**
  A CAN receive update occurs within 100 ms

* **Fault condition:**
  No healthy CAN receive update occurs for more than 100 ms

* **System response:**
  `FAULT_CAN` is set in the fault bitmask

This timing prevents the system from using stale communication data while still allowing short timing variations.

## CAN Behavior

The CAN fault logic follows a simple receive-health requirement:

* **Healthy state:**
  CAN messages are being received regularly, and `canLatestHealthyStateTime` is updated

* **Overdue state:**
  The time since the last healthy CAN receive exceeds `CAN_FAULT_TIME_THRESHOLD_MS`

* **Fault response:**
  `Faults_SetFault(FAULT_CAN)` is called

* **Recovered state:**
  CAN messages resume within the expected window, and `Faults_ClearFault(FAULT_CAN)` is called

This allows the system to automatically detect both communication loss and recovery.

## Fault Handling & Diagnostics

The CAN fault is handled through the shared vehicle fault system.

| Condition        | Fault Action                   |
| ---------------- | ------------------------------ |
| CAN age ≤ 100 ms | `Faults_ClearFault(FAULT_CAN)` |
| CAN age > 100 ms | `Faults_SetFault(FAULT_CAN)`   |

Because the CAN fault uses the shared bitmask system, it can be checked alongside other faults such as APPS, BSE, and watchdog-related faults.

## Testing and Validation

The CAN fault logic can be validated by observing whether the fault bit changes correctly during normal and failed communication.

Testing should include:

* Confirming `canLatestHealthyStateTime` updates on every CAN receive
* Verifying `FAULT_CAN` stays cleared during normal CAN traffic
* Stopping CAN messages and confirming `FAULT_CAN` is set after 100 ms
* Restoring CAN messages and confirming `FAULT_CAN` clears
* Checking that the CAN fault appears correctly in the shared fault bitmask

This confirms that the system correctly detects stale CAN communication and reports it through the vehicle fault system.
