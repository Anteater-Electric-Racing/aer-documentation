# Thermal Derating Module

Author: Atharva Rao 

## Purpose

Prevent the BMS and AIRs from triggering a hard reset by reducing torque and current limits before hardware-enforced thresholds are reached. Derating is applied across the battery, motor, and inverter, which takes effect on every torque command sent to the DTI.

---

## How It Works

`VCU_Derate(float temperature)` returns a scalar factor in [0.2, 1.0] using piecewise linear interpolation between two temperature thresholds:

| Constant | Value | Meaning |
|---|---|---|
| `TEMP_START` | 70 °C | Derating begins |
| `TEMP_MAX` | 100 °C | Maximum derating (factor = 0.2) |

```
factor = 1.0 − (0.8) × (temp − TEMP_START) / (TEMP_MAX − TEMP_START)
```

Below `TEMP_START` the factor is 1.0 (no derating). Above `TEMP_MAX` the input is clamped and the factor floors at 0.2. 

---

## Integration 

Three factors are computed independently each VCU loop iteration and the minimum is used:

```cpp
float batteryFactor  = VCU_Derate(BMS_GetOrionData()->highTemp);
float motorFactor    = VCU_Derate(DTI_GetDTIData()->motorTemp);
float inverterFactor = VCU_Derate(DTI_GetDTIData()->controllerTemp);

float smallestFactor = min(batteryFactor, min(motorFactor, inverterFactor));

DTI_SetDCLimits(60.0 * smallestFactor, -2.0);
DTI_SetACLimits(150.0 * smallestFactor, -20.0);
DTI_SendAccelCommand(targetTorque * smallestFactor);
```

Both the CAN-sent current limits and the torque command are scaled by the same factor, so all three limit layers (BMS config, DTI config, CCM CAN override) are adjusted together.

---

## Limitations

- A single linear ramp is used for all three thermal sources, no per-component curves.
- All temperature thresholds are placeholder (TEMP_START, TEMP_MAX, MIN_FACTOR) which require adjustments.