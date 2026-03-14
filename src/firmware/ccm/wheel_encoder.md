Author: Pranav Kocharlakota

# Wheel Encoder

## Purpose

The wheel encoder module measures how fast a wheel is spinning, reporting the result in **RPM (revolutions per minute)**. It works by detecting a gear attached to the wheel — each time a tooth on that gear passes a sensor, the module takes note of the timing and uses that to figure out how fast the wheel is turning.

## How It Keeps Track of Speed

The module is always listening in the background. Whenever a gear tooth passes the sensor, it quietly records the exact time of that event and compares it to the previous one. That time gap is the key piece of information used to calculate RPM.

This listening happens automatically and independently from everything else the system is doing, so it never misses a pulse even while the rest of the program is busy.

If no gear tooth passes the sensor for more than **half a second**, the module assumes the wheel has stopped and reports **0 RPM**. This prevents the system from showing a stale, outdated speed reading when the wheel is no longer moving.

## Calculating RPM
 
Once the time between two pulses is known, the speed is calculated by figuring out how long a full rotation would take at that rate — accounting for the number of teeth on the gear — and converting that into revolutions per minute.
 
```
RPM = 60,000,000 / (Δt × N)
```
 
Where:
- **Δt** = time between the last two pulses (in microseconds)
- **N** = number of teeth on the gear
 
A gear with more teeth gives more frequent pulses, which means better accuracy at low speeds. A gear with fewer teeth gives fewer pulses, which is fine at higher speeds but less precise when the wheel is moving slowly.

## Setup and Ongoing Use

The module has two distinct phases:

**1. Startup**
When the system first powers on, the module sets up the sensor pin and tells the hardware to start listening for gear tooth signals automatically. This only needs to happen once.

**2. Running**
While the system is operating, the module should be checked regularly (many times per second is ideal). Each check looks at the most recent pulse timing, applies the RPM calculation, and updates the current speed reading. Anything in the system that needs the wheel speed can then simply ask for the latest value.

## Summary of Constants

| Setting | What It Controls |
|---|---|
| **Sensor pin** | Which physical pin on the board the sensor is connected to |
| **Gear teeth** | How many teeth are on the encoder gear |
| **Timeout** | How long to wait without a pulse before reporting 0 RPM (default: 0.5 seconds) |
