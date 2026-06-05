Authors: Anushree Godbole, Kenneth Dao
# Watchdog Timer (WDT) Implementation
## Overview
The Watchdog Timer (WDT) subsystem is implemented to ensure the reliable operation of critical vehicle control software. Its primary function is to detect software stalls or timing violations in key tasks and enforce recovery through a system reset.

This implementation operates at the thread level and monitors the execution health of four critical threads:
- ADC Thread
- Main Thread
- VCU Thread
- CAN Thread

If any thread fails to update within a defined threshold, the watchdog is no longer fed, allowing the Teensy's hardware watchdog to reset the system.

## System Architecture
The watchdog does not maintain persistent fault states; all checks are performed on live thread timing data during each evaluation cycle.

The WDT system consists of:
1. Hardware Watchdog Peripheral (via `Watchdog_t4`)
2. FreeRTOS Monitoring Task (`threadWDT`)
3. Timing Tracking Variables (`*_last_run_tick`)
4. Bitmask-Based Overdue Thread Detection

Each monitored thread updates its own timestamp (`*_last_run_tick`) whenever it executes successfully. The watchdog task periodically checks these timestamps to determine whether all critical threads are executing as expected.

## Overdue Thread Detection Strategy
The system uses a time-based detection approach:
- Each thread must update within a defined threshold 
- The watchdog task runs periodically to evaluate thread health
- A thread is considered overdue if the elapsed time since its last heartbeat exceeds the threshold

During each watchdog evaluation cycle, a bitmask is used to identify which threads are overdue. This allows multiple thread failures to be tracked simultaneously and enables easy scalability for additional monitored components.

## Bitmask Representation

Overdue threads are encoded using a bitmask:

| Thread | Bit | Value |
|----------|-----|------|
| ADC      | 0   | 0b0001 |
| Main     | 1   | 0b0010 |
| VCU      | 2   | 0b0100 |
| CAN      | 3   | 0b1000 |

- `WDT_REQUIRED_MASK = 0b0000` indicates that no threads are overdue and the system is healthy  
- If any thread is marked as overdue, the watchdog is not fed.

## Timing Relationship: Detection vs. Reset

The watchdog system uses two distinct timing parameters that serve different purposes:

- **100 ms (Thread Timing Threshold):**  
  Maximum allowable time between thread updates before a thread is considered overdue.

- **1 second (Hardware Watchdog Timeout):**  
  Defines how long the system can go without being fed before the hardware watchdog triggers a reset of the Teensy 4.1.

## Watchdog Behavior

The watchdog operates on a strict all-threads-healthy requirement:

- **Healthy System:**  
  All monitored threads update within their thresholds → watchdog is fed continuously   

- **Overdue Thread Detected:**  
  One or more threads exceed their timing threshold → watchdog is not fed

- **System Response:**  
  If the watchdog is not fed within its configured timeout (1 second), the hardware automatically resets the Teensy 4.1

This ensures that any persistent software failure results in a full system restart.


## Diagnostics and Logging

When an overdue thread is detected, the watchdog logs the condition via serial output:

| Condition | Message |
|----------|--------|
| ADC Thread overdue | "WDT: ADC thread overdue" |
| CAN Thread overdue | "WDT: CAN thread overdue" |
| Main Thread overdue | "WDT: Main thread overdue" |
| VCU Thread overdue | "WDT: VCU thread overdue" |