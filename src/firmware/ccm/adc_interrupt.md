Author: Pranav Kocharlakota and Wilson Nguyen

# Automated Analog Data Pipeline // Anteater Electric Racing, 2025

## Functional Architecture
The sensor acquisition subsystem replaces a classic manual analog reading strategy—where the processor would explicitly halt execution to read pins one by one—with a hardware-driven, zero-overhead pipeline. When triggered by hardware timers, the system automates sensor sampling, sequences the channel configuration, and transfers the data directly to memory using Direct Memory Access (DMA). 

This approach eliminates CPU polling overhead entirely. The processor remains completely unaware of analog conversions until an entire frame of critical vehicle data (comprising suspension linpots, throttle, and brake sensors) is cleanly packaged in RAM.

## Integrated Hardware Sequencing
The subsystem links several internal microcontroller blocks into an automated pipeline that operates independently of the core processor:

1. **PIT (Periodic Interrupt Timer):** Fires hardware trigger signals at a fixed rate of 1kHz. PIT0 and PIT1 are intentionally phase-offset by 50 microseconds to stagger conversion waves and prevent electrical noise or power dips from corrupting consecutive samples.
2. **XBAR (Cross-bar Switch):** Routes the hardware pulses directly from the PIT timers to the ADC_ETC hardware trigger slots without utilizing interrupt service routines.
3. **ADC_ETC (ADC Error To Control):** Configured to manage two distinct conversion chains containing 4 sensor channels each. Upon receiving an XBAR pulse, it automates channel rotation, commands the hardware ADC to convert the pins, and sequences results into a dense register window.
4. **DMA & Ping-Pong Buffers:** As soon as the second chain finishes converting, the DMAMUX signals the DMA controller. The DMA engine executes a minor-loop copy, moving the entire 12-word data frame out of the ADC_ETC registers and dropping it into one half of a dual ping-pong buffer in RAM.

## Memory Management & Data Processing
To ensure the decoder task never reads half-written or corrupted data, the subsystem implements a 32-byte aligned ping-pong buffer strategy (`adc_dma_buffer[2][8]`). 

While the DMA engine is actively populating `active_buffer`, the processing thread is safely decoding the static data stored in `ready_buffer`. Because the processor utilizes a data cache that could cause it to read stale memory state, the `DMA_ISR` explicitly clears the CPU cache line using `arm_dcache_delete` immediately after a transfer finishes. 

Once the cache is cleared and the buffer indexes flip, the handler issues a fast FreeRTOS task notification to wake the `threadADC` decoder task. The decoder reads the raw values out of the completed buffer and immediately forwards them to their respective device processing modules:

* **Chain 0 (Linear Pots 1–4):** Dispatched via `ShockTravelUpdateData()` to track suspension dynamics.
* **Chain 1 (APPS 1–2 & BSE 1–2):** Dispatched via `APPS_UpdateData()` and `BSE_UpdateData()` to feed torque commands and brake plausibility checks into the VCU core.

## Execution Constraints
The `DMA_ISR` runs at maximum hardware priority, blocking all normal vehicle thread execution until it exits. Doing any actual sensor processing, calibration, or mathematical scaling inside this handler would stall the VCU and risk missing timing deadlines for high-priority tasks like the CAN transmit loops. 

The handler is strictly confined to clearing flags, swapping the active buffer pointer, invalidating the CPU cache, and unblocking the decoder thread. The actual parsing and downstream distribution of the sensor data happens safely at normal priority inside the decoupled `threadADC` loop.

## Validation Procedure

### Protocol 1: Standard Verification
**Goal:** Confirm ADC data is continuously sampling, transferring via DMA, and feeding the device processing modules correctly without cache corruption.

1. Flash the VCU and power on the low-voltage master switch with telemetry modules active.
2. Open the Serial Monitor and confirm that the VCU outputs `"Beginning adc thread"` and does not trip the hardware watchdog (`wdt.h`).
3. View live data plots via the telemetry terminal or a connected laptop. Verify that moving the physical throttle pedal, pressing the brake pedal, and compressing the suspension linpots displays smooth, real-time value changes.
4. If sensor data updates at a continuous 1kHz rate, signals track movement linearly, and no DMA error flags lock up the thread, the pipeline is functioning correctly. Stop here.

### Protocol 2: Core Isolation & Timing Diagnostics (Fallback)
**Goal:** Identify why sensor values appear frozen, display erratic jumps, or output corrupted zeroes.

1. Inspect the raw variables in a debug session. If data changes in the `adc_dma_buffer` array but fails to propagate to the `APPS` or `BSE` modules, confirm that `adcDecoderTaskHandle` is successfully assigned during initialization and that the task is not being starved by a higher-priority loop.
2. Erratic data jumps or partial frames usually point to cache incoherency. Ensure that the `adc_dma_buffer` is properly assigned to the uninitialized, non-cacheable RAM section (`.noinit.$RAM2`) and that `arm_dcache_delete` is being called *before* the buffer is processed by the thread.
3. If the conversion chains overlap or cross-talk occurs between sensors, check the `PHASE_OFFSET_US` value. Increase the timing offset between PIT0 and PIT1 to give the hardware analog front-end more time to settle between channel sequences.