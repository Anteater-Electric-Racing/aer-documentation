# ADC Polling to Interrupts
By Wilson Nguyen
## Overview

Currently, the CPU is polling the ADCs on the Teensy, which blocks the CPU until the ADC task is done. This is synchronous I/O. However, this wastes CPU compute and power because while the CPU is waiting for the I/O event to finish, it is not making progress on other tasks.

The new method utilizes the Teensy's DMA (direct memory access) hardware and interrupts to efficiently move the ADC data from the ADC registers into a buffer in RAM via DMA. DMA is a specialized hardware mechanism that allows peripheral devices to directly transfer data to or from RAM without CPU involvement. Once the buffer is full, an interrupt signal is generated and the ISR uses a direct-to-task notification to wake the ADC task. The ADC task writes the data from the buffer into a struct, which the CAN task will then transmit over the CAN bus.

The goal is to free the CPU from having to manually poll and read the ADCs and allow it to spend time progressing with other tasks.

## Architecture

### 1. PIT (Periodic Interrupt Timer)

- Generates a trigger pulse (`PIT_TRIGGER0` and `PIT_TRIGGER1`) every 1 ms / 1 kHz.

### 2. XBAR

- Internal signal router.
- Routes `PIT_TRIGGER0` -> `XBAR1_OUT103` -> `ADC_ETC_TRIG00`.
- Routes `PIT_TRIGGER1` -> `XBAR1_OUT104` -> `ADC_ETC_TRIG01`.

### 3. ADC ETC (External Trigger Controller)

- Receives the XBAR output signal.
- Starts the chain to sample ADC channels in a specific order.
- Sends trigger pulses to ADC to rotate channels.
- Gives the ADC hardware trigger sequence to the ADC driver.

### 4. ADC1

- 1 or 0 depending on whether you are reading the documentation or the code, but it is the first ADC.
- Performs the actual analog-to-digital conversions at a 12-bit resolution.
- Passes conversion results back to the registers.

### 5. ADC ETC Result Registers

- Stores completed ADC results.
- Triggers a DMA request upon conversion completion.

### 6. DMA MUX

- Routes ADC ETC's DMA request to DMA channel 23.

### 7. DMA

- Moves data from ADC ETC result registers into a buffer in RAM2.
  - We use RAM2 because it is a general-purpose on-chip RAM and the DMA bus can freely move data to and from RAM2.

### 8. DMA Interrupt

- After the ready buffer is complete, the interrupt fires and the DMA ISR (interrupt service routine) executes.
- The ISR uses ping-ponging to prevent any read-write conflicts:
  1. First, we set the `ready_buffer` (buffer the CPU will read from) to the `active_buffer` (buffer the DMA just wrote to).
  2. Then, we clear the old ADC data in the CPU cache because the CPU needs to read fresh data.
  3. Next, we swap the `active_buffer` to the other buffer in `adc_dma_buffer`.
  4. After that, we clear the `active_buffer` so that the DMA can safely write to it.
  5. Finally, we wake the ADC decoder task via a direct-to-task notification.

### 9. ADC Task

- Wakes up and passes the data to the respective sensor peripherals for proper scaling and struct storage.

## Sensor to ADC Channel Mapping

| Sensor | Analog pin | Pad name | ADC1 channel |
|---|---:|---|---|
| Linear potentiometer 1 | A9 | `GPIO_AD_B1_09` | `ADC1_IN14` |
| Linear potentiometer 2 | A8 | `GPIO_AD_B1_08` | `ADC1_IN13` |
| Linear potentiometer 3 | A7 | `GPIO_AD_B1_11` | `ADC1_IN0` |
| Linear potentiometer 4 | A6 | `GPIO_AD_B1_10` | `ADC1_IN15` |
| APPS1 | A5 | `GPIO_AD_B1_00` | `ADC1_IN5` |
| APPS2 | A4 | `GPIO_AD_B1_01` | `ADC1_IN6` |
| BSE1 | A3 | `GPIO_AD_B1_06` | `ADC1_IN11` |
| BSE2 | A2 | `GPIO_AD_B1_07` | `ADC1_IN12` |

## Testing / Verification

Testing has not been fully completed. However, a full testing plan would be to set up a test bench consisting of the Teensy and, if possible, all 8 sensors connected to the Teensy. An initial test plan would be to connect a PSU to the Teensy and supply a voltage of 0-3.3 V and test if we are getting the correct readings.
