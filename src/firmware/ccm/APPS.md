# APPS Module Documentation
**Anteater Electric Racing, 2025**

---

## Overview

The **APPS (Accelerator Pedal Position Sensor)** module processes two independent pedal sensors to determine driver throttle input.

It performs:
- Signal filtering
- Voltage conversion from ADC values
- Pedal percentage mapping (0.0 → 1.0)
- Sensor fault detection
- Brake–throttle plausibility checks using BSE data

The module is designed as a **safety-critical component**, ensuring redundancy and preventing unsafe operating conditions.

---

## System Data Flow

The APPS module receives raw ADC values from an external ADC (ADS1115) and processes them into usable throttle values.

### Data Pipeline

ADS1115 ADC -> Raw ADC Counts -> Low-pass filter ->Voltage conversion -> Percentage mapping (0–1) -> Fault detection -> Throttle output

---

## Input Sources

Two independent sensors are used:

| Sensor | Voltage Range | Description |
|--------|--------------|-------------|
| APPS1  | 0–5V         | Primary pedal sensor |
| APPS2  | 0–3.3V       | Secondary redundant sensor |

---

## Internal Data Structure

```cpp
typedef struct {
    float appsReading1_Percentage;
    float appsReading2_Percentage;

    float appsReading1_Voltage;
    float appsReading2_Voltage;

    float apps1RawReading;
    float apps2RawReading;
} APPSData;
```

---

## Initialization

### `APPS_Init()`

Initializes all state variables and sets filter parameters.

- Sets all readings to zero
- Configures low-pass filter:

```cpp
appsAlpha = COMPUTE_ALPHA(100.0F);
```

(100 Hz cutoff frequency)

---

## Main Update Function

### `APPS_UpdateData(raw1, raw2)`

Processes new ADC data from both sensors.

---

## Processing Steps

### 1. Low-Pass Filtering

```cpp
filtered = previous + alpha * (new - previous);
```

---

### 2. ADC to Voltage Conversion

```cpp
voltage = raw * ADS_VOLTS_PER_BIT * 1.25;
```

---

### 3. Voltage to Percentage Mapping

```cpp
APPS1: 0–5V   → 0–1
APPS2: 0–3.3V → 0–1
```

---

### 4. Clamping

Voltage and percentage values are clamped within safe bounds.

---

## Fault Detection

### Electrical Fault

Triggered when:
- Voltage out of range
- Sensors disagree beyond threshold

### Time Requirement

Fault must persist longer than:

APPS_FAULT_TIME_THRESHOLD_MS

---

### Sensor Disagreement

difference = |APPS1 - APPS2|

If:

difference > APPS_IMPLAUSABILITY_THRESHOLD

→ Fault triggered

---

## Brake–Throttle Plausibility

Uses BSE readings.

Triggered when:

Throttle > threshold AND Brake > threshold

Reset when:

Throttle < reset threshold

---

## Output Functions

Average throttle:

APPS = (APPS1 + APPS2) / 2

---

## Safety Design

- Dual sensor redundancy
- Noise filtering
- Time-based fault validation

---

## Important Notes

- Mixed 5V and 3.3V sensors require calibration
- Scaling factor (1.25) affects voltage conversion
- LINEAR_MAP does not clamp automatically

---

## Summary

The APPS module ensures safe, reliable throttle input by validating and processing redundant sensor data before it is used by the motor control system.
