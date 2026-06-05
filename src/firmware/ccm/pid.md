# PID Controller Module
Author: [Your Name]

## Purpose
A generic, reusable PID controller for any closed-loop control application. Integral windup is clamped to configurable limits set at initialization.

---

## How It Works

`computePID()` implements a standard discrete PID:

| Term | Formula |
|---|---|
| Error | `e = setPoint − input` |
| Integral | `∫ e · dt`, clamped to [`windUpLimitMin`, `windUpLimitMax`] |
| Derivative | `(e − e_prev) / dt` |
| Output | `Kp·e + Ki·∫e + Kd·de/dt` |

- `dt` is computed from FreeRTOS tick counts via `xTaskGetTickCount() / configTICK_RATE_HZ`.
- Anti-windup: integral is hard-clamped each iteration to the limits set in `pidConfig()`.
- Output is unbounded — the caller is responsible for clamping to actuator limits.

---

## Integration

```cpp
PID myController;
pidConfig(&myController, WINDUP_MAX, WINDUP_MIN);

// In control loop:
float output = computePID(&myController, setPoint, input, KP, KI, KD);
```

`pidReset()` zeroes the integral, previous error, and timestamp — use it when re-enabling a controller after a fault or mode switch to prevent state carryover.

---

## Limitations
- Gains (`Kp`, `Ki`, `Kd`) are passed per-call rather than stored in the struct — caller must ensure consistent gains across iterations.
