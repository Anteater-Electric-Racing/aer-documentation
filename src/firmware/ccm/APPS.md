<div align="center">

# APPS Module Overview
#### Anteater Electric Racing

</div>

---

## Purpose
Process **two pedal sensors** and output a safe, normalized throttle value.

- Converts raw ADC → **pedal % (0–1)**
- Ensures both sensors **agree**
- Detects **faults (electrical + plausibility)**
- Prevents **brake + throttle simultaneously**

---

## High Level Flow

Raw ADC → Filter → % Mapping → Voltage Check → Faults → Output

---

## Update Function

### `APPS_UpdateData(raw1, raw2)`

1. **Filter Inputs**
- Apply low-pass filter (100 Hz)
- Smooths noise and spikes
- This helps keep the pedal signal stable instead of jumping around from tiny electrical fluctuations

---

2. **Convert to Pedal %**

- Uses calibrated raw values:
```
rest → 0%
full → 100%
```

```
apps1 = LINEAR_MAP(raw1, REST1, FULL1, 0 → 1)
apps2 = LINEAR_MAP(raw2, REST2, FULL2, 0 → 1)
```

- This means the code is mainly built around **real measured ADC values** from the pedal, not just ideal voltages
- That makes calibration easier because the mapping matches the actual installed hardware

---

3. **Clamp Values**
- Ensure:
```
0 ≤ APPS ≤ 1
```

- If the sensor goes slightly below rest or above full travel, the final output is still kept in a safe range

---

4. **Convert to Voltage**
Used for fault detection only:

```
voltage = ADC_VALUE_TO_VOLTAGE(raw)
```

- The code still converts raw readings into voltage so it can check for open-circuit or short-circuit behavior
- So percentage is based mainly on **raw ADC calibration**, while voltage is used mainly for **electrical safety checks**

---

5. **Voltage Safety Check**

APPS1 (3.3V):
```
0.33V → 2.97V (fault range)
```

APPS2 (5V):
```
0.5V → 4.5V (fault range)
```

- If a sensor voltage goes outside these ranges for too long, the system raises an APPS fault
- This helps catch wiring issues, sensor faults, or readings that are no longer physically believable

---

6. **Sensor Agreement Check**

```
difference = |APPS1 - APPS2|
```

If:
```
difference > 0.1 (10%)
```
→ FAULT

- Since the pedal uses two sensors, they should track each other closely
- If they disagree too much, the system treats that as unsafe

---

7. **Brake Plausibility Check**

If:
```
Throttle > 25%
AND
Brake > threshold
```

→ FAULT

Reset when:
```
Throttle < 5%
```

- This prevents the car from accepting heavy throttle while strong braking is also being detected

---

## Output

```
APPS = (APPS1 + APPS2) / 2
```

- The final pedal command is the average of both sensors
- This gives one clean throttle value to pass into the rest of the vehicle logic

---

## ADS1115 Integration

The APPS code is fed by the **ADS1115 external ADC**.

In `adc.cpp`, the ADS reads the two pedal channels and passes them into:

```cpp
APPS_UpdateData(raw1, raw2);
```

So the flow is:

```text
Pedal sensor → ADS1115 → raw ADC counts → APPS_UpdateData()
```

### Why use the ADS1115?

The ADS1115 gives **16-bit readings**, which means it can measure the sensor signal with much finer resolution than a typical 12-bit onboard ADC.

That helps because:

- **higher resolution** gives smaller voltage steps between readings
- **better precision** makes pedal calibration cleaner
- **less quantization error** means smoother percentage mapping
- **less noise sensitivity** makes the signal easier to filter and trust

In simple terms, the ADS gives the APPS module a more detailed picture of what the pedal is doing.

### Why that matters for APPS

Since APPS is safety-critical, better signal quality is valuable:
- small pedal changes are easier to detect
- the filtered signal is smoother
- the two sensors can be compared more accurately
- fault checks become more reliable

So while the APPS logic itself is mainly based on **filtered raw counts and calibration**, the ADS1115 improves the quality of those raw counts before APPS ever sees them.

---

## Summary

APPS:
- Converts pedal → throttle
- Verifies sensor agreement
- Blocks unsafe conditions

The ADS1115 helps by giving **high-resolution 16-bit sensor readings** with better accuracy and lower effective noise, making the APPS signal cleaner and more trustworthy before torque is commanded.
