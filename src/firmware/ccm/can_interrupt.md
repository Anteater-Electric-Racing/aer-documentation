Author: Pranav Kocharlakota

# CAN Interrupt Handler

## Overview

`CAN_RxInterruptHandler` replaces a constantly polling system — where the processor would repeatedly check if a message had arrived — with one that only runs when a message actually arrives. When triggered, it takes the message's **ID** and **data**, bundles them into a `CANMessage_t` struct, and drops it into a queue. This frees the processor to do other work between messages rather than burning cycles checking for them.

## The Queue

The queue holds up to 10 messages at a time. If a message arrives when the queue is full, it is dropped and the **overflow counter** increments. However, messages should be cleared from the queue faster than the queue can be populated so the overflow counter acts as a safety and testing mechanism. If a task was waiting on the queue, the handler wakes it up immediately after adding the message.

## Why It's Kept Short

The handler runs at a higher priority than everything else, blocking all other execution while it runs. Doing anything beyond packaging and queuing would stall the system. The actual processing of each message happens separately, at normal priority.

## Testing Plan
 
### 1. Normal Operation
**Goal:** Confirm CAN messages are being received and processed correctly with no overflow.
- Power on the system with all devices connected to the CAN bus.
- Open the Serial Monitor and confirm that CAN messages are being printed — precharge data is readable by the CCM and inverter commands are being received.
- Monitor the Serial Monitor for the duration of the test. If no overflow message appears, the handler is working correctly.
- Cross-check the received values against known expected values (e.g., correct voltage, temperature, SOC) to confirm data is not being corrupted in transit.
- If no overflow message is printed and values look correct, the code is functioning as expected. Stop here.
 
### 2. Overflow Debugging (Fallback)
**Goal:** Identify why overflow messages appeared in step 1.
- Open CANalyzer and connect to the bus. Log all incoming messages and note the total message rate per second and which IDs are transmitting. Ensure that it is only essential devices and all are transmitting at normal rates.
- The size of the queue may be too small increase it by 10 until overflow is gone. The ideal size of the queue is between 10 and 30.
- Do not increase the queue by huge increments. This is because a large queue takes more memory and more importantly a large queue paints over possible issues with transmission rates of other devices.
