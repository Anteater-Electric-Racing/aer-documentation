# Thermal Regulation Module
Author: [Your Name]

## Purpose
Maintain safe operating temperatures for the coolant pumps and radiator fan using closed-loop PID control, adjusting PWM duty cycles in response to live temperature data before hardware thresholds are reached.

---

## How It Works

`thermal_regulate()` runs three independent PID controllers — one per actuator — against a shared temperature input. Each controller returns a normalized scalar in [0.0, 1.0] representing cooling effort, which maps to a PWM duty cycle.

### `computePID()` Behavior

| Term | Formula |
|---|---|
| Error | `e = input − setPoint` |
| Integral | `∫ e · dt`, accumulates only when output ∈ (0, 1) |
| Derivative | `(e − e_prev) / dt` |
| Output | `Kp·e + Ki·∫e + Kd·de/dt`, clamped to [0, MAX_OUTPUT], then divided by MAX_OUTPUT |

- `dt` is computed from FreeRTOS tick counts via `xTaskGetTickCount() / configTICK_RATE_HZ`.
- If `input < setPoint`, the controller resets state and returns 0 — no cooling below threshold.
- Anti-windup: integral only accumulates when the actuator is unsaturated.

### Output Inversion

Higher cooling effort requires a higher duty cycle, so the PID output is inverted before writing:

```
pwmDuty = DUTY_CYCLE_MAX × (1 − pidOutput)
```

---

## Integration

```cpp
float temp = max(DTI_GetDTIData()->controllerTemp, DTI_GetDTIData()->motorTemp);

float fanOutput   = computePID(&fan,   FAN_THRESHOLD, temp, FAN_KP,   FAN_KI,   FAN_KD);
float pump1Output = computePID(&pump1, FAN_THRESHOLD, temp, PUMP1_KP, PUMP1_KI, PUMP1_KD);
float pump2Output = computePID(&pump2, FAN_THRESHOLD, temp, PUMP2_KP, PUMP2_KI, PUMP2_KD);

analogWrite(FAN_PIN,   DUTY_CYCLE_MAX * (1 - fanOutput));
analogWrite(PUMP1_PIN, DUTY_CYCLE_MAX * (1 - pump1Output));
analogWrite(PUMP2_PIN, DUTY_CYCLE_MAX * (1 - pump2Output));
```

PID states are `static` locals in `thermal_regulate()` and persist across calls. PWM is initialized at 25 kHz (`ANALOG_WRITE_FREQUENCY`) in `thermal_Init()`. `thermal_forceOn()` and `thermal_forceOff()` bypass PID for fault states or bench testing.

---

## Limitations
- All gain constants are placeholder values requiring physical tuning.
