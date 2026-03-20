Authors: Anushree Godbole, Kenneth Dao
# Watchdog Timer (WDT) Implementation
## Overview
The Watchdog Timer (WDT) subsystem is implemented to ensure the reliable operation of critical vehicle control software. Its primary function is to detect software stalls or timing violations in key tasks and enforce recovery through a system reset.

This implementation monitors the execution health of three safety-critical subsystems:
- Brake System Encoder (BSE)
- Accelerator Pedal Position Sensors (APPS)
- **Traction Control (planned, not yet implemented)**

If any subsystem fails to update within a defined time threshold, the watchdog is no longer serviced, allowing the microcontroller’s hardware watchdog to reset the system.

## System Architecture
The WDT system consists of:
1. Hardware Watchdog Peripheral (via `Watchdog_t4`)
2. FreeRTOS Monitoring Task (`WDT_Update_Task`)
3. Timing Tracking Variables (`*_last_run_tick`)
4. Bitmask-Based Fault Detection

Each monitored subsystem updates a shared timestamp (`*_last_run_tick`) whenever it executes successfully. The watchdog task periodically checks these timestamps to determine system health.

## Fault Detection Strategy
The system uses a time-based fault detection approach:
- Each subsystem must update within a defined fault threshold 
- The watchdog task runs periodically to evaluate subsystem health
- A subsystem is considered faulty if its update interval exceeds the threshold

Fault states are internally represented using a bitmask, allowing multiple subsystem failures to be tracked simultaneously and enabling easy scalability for additional monitored components.

## Fault Representation

Faults are encoded using a bitmask:

| Subsystem | Bit | Value |
|----------|-----|------|
| BSE      | 0   | 0b01 |
| APPS     | 1   | 0b10 |

- `WDT_REQUIRED_MASK = 0b00` represents no flags and a fully healthy system  
- If any fault bit is set, the watchdog is not serviced, allowing timeout  

## Timing Relationship: Detection vs. Reset

The watchdog system uses two distinct timing parameters that serve different purposes:

- **100 ms (Fault Threshold for APPS and BSE):**  
  Defines how frequently the system checks the health of the subsystems and detects timing violations  

- **1 second (Watchdog Timeout):**  
  Defines how long the system can go without being serviced before the hardware watchdog triggers a reset  

A fault must persist across multiple monitoring cycles before resulting in a watchdog timeout, ensuring both responsiveness and stability.

## Watchdog Behavior

The watchdog operates on a strict all-systems-healthy requirement:

- **Healthy system:**  
  All monitored subsystems update within their thresholds → watchdog is continuously serviced  

- **Fault condition:**  
  One or more subsystems exceed their timing threshold → watchdog is not serviced  

- **System response:**  
  If the watchdog is not serviced within its configured timeout (1 second), the hardware automatically resets the microcontroller  

This ensures that any persistent software failure results in a full system restart.


## Fault Handling & Diagnostics

If a fault is detected, the system logs the condition via serial output:

| Condition | Message |
|----------|--------|
| BSE overdue | "WDT: BSE update overdue" |
| APPS overdue | "WDT: APPS update overdue" |
| Both overdue | "WDT: BSE and APPS updates overdue" |