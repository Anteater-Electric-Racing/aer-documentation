
<div align="center">

# Shock Sensor Integration
#### Anoop Koganti

</div>

## Purpose
Design a function that converts **4 shock sensor ADC readings** into usable **shock travel values in millimeters** for telemetry and suspension analysis.

The shock sensors are **linear potentiometers** whose signal sits close to **5 V when the shock is fully extended**.
When the suspension compresses, the voltage only decreases slightly.

The total useful signal range is approximately:

**5.00 V → 4.86 V**

So the **actual change is only about 0.14 V**.

Because the change is very small relative to the 5 V baseline, the code treats **the difference from 5 V** as the meaningful signal and scales that difference to the full mechanical travel of the shock.

---

## Algorithm Overview

### Initial Function
- Reset all stored shock sensor values
- Clear:
  - raw ADC readings
  - converted voltages
  - shock travel values in mm

This ensures startup begins with known values instead of stale sensor data.

---

### Update Function

1. **Read 4 ADC values**
   - Inputs come in as `rawReading1` through `rawReading4`

2. **Invert ADC readings**
   - Each reading is converted using:

```cpp
abs(4095 - rawReading)
```

- The ADC is 12-bit, meaning values range from **0 to 4095**.
- Because the sensor voltage decreases during compression, the raw ADC value behaves opposite of the desired interpretation of travel.
- Subtracting from 4095 flips the scale so that **increasing compression produces increasing processed signal values**.

3. **Convert ADC counts to voltage**

```cpp
ADC_VALUE_TO_VOLTAGE(...)
```

This converts the processed ADC value into volts.

4. **Use `abs()` on voltage**

```cpp
abs(ADC_VALUE_TO_VOLTAGE(...))
```

This ensures the voltage value is always non‑negative.
It mainly acts as a safety guard against unexpected sign behavior.

5. **Map voltage difference to shock travel**

The shock signal only changes by **0.14 V** across its usable range:

**5.00 V → 4.86 V**

The code maps that small change to the full suspension travel using:

```cpp
(voltage / 0.14f) * SHOCK_TRAVEL_MAX_MM
```

Conceptually this means:

| Voltage | Shock Travel |
|-------|-------|
| 5.00 V | 0 mm |
| 4.93 V | ~50% travel |
| 4.86 V | max travel |

6. **Clamp output**

The computed travel is limited using:

```cpp
constrain(value, 0.0f, SHOCK_TRAVEL_MAX_MM)
```

This prevents invalid sensor readings from producing impossible travel values.

---

## Why `abs()` is used

### 1. Flipping the ADC scale

```cpp
abs(4095 - rawReading)
```

The sensor begins near **5 V when fully extended**, so the ADC reading starts close to the top of the ADC range.

As the shock compresses:

- Voltage drops slightly
- ADC value drops slightly

By subtracting from **4095**, the scale is mirrored so the processed signal **increases with compression**.

The `abs()` ensures the result stays positive.

### 2. Protecting voltage conversion

```cpp
abs(ADC_VALUE_TO_VOLTAGE(...))
```

This simply ensures the converted voltage value cannot become negative due to any intermediate math behavior.

The important calibration step is not `abs()` itself but recognizing that the **signal range is only 0.14 V below 5 V**.

---

## Implementation Details

### `void Shock_Init()`

Initializes all stored linear potentiometer data to zero.

Values cleared:

- raw ADC readings
- converted voltages
- shock travel values

This function runs once during startup.

---

### `void ShockTravelUpdateData(...)`

Processes all four shock sensor ADC readings.

Steps:

1. Flip ADC readings using `4095 - rawReading`
2. Convert processed ADC values to voltages
3. Interpret the small **0.14 V drop from 5 V** as suspension compression
4. Scale that voltage change into millimeters
5. Clamp output values within the physical travel range

---

### `Linpot_GetData()`

Returns a pointer to the shared `linPots` data structure.

Other modules can read:

- raw processed sensor values
- voltages
- shock travel in millimeters

This allows telemetry and future control algorithms to access suspension data.

---

## Example

### Example input

If the sensor is fully extended:

```
rawReading ≈ 4095
```

After inversion:

```
4095 - 4095 = 0
```

Voltage becomes near **5 V baseline**, so travel ≈ **0 mm**.

---

If the voltage drops by **0.07 V**:

```
5.00 V → 4.93 V
```

That represents roughly **half of the total 0.14 V span**, so:

```
travel ≈ 0.5 × SHOCK_TRAVEL_MAX_MM
```

---

# TESTING PLAN

Method Signature:

```
ShockTravelUpdateData(uint16_t rawReading1,
                      uint16_t rawReading2,
                      uint16_t rawReading3,
                      uint16_t rawReading4)
```

Constants:

```
ADC max count = 4095
voltage span = 0.14 V
minTravel = 0
maxTravel = SHOCK_TRAVEL_MAX_MM
```

---

## TestCase 1 (Initialization)

Call:

```
Shock_Init()
```

Expected behavior:

- All voltages = 0
- All raw readings = 0
- All shock travel values = 0 mm

---

## TestCase 2 (Fully Extended)

Inputs correspond to ~5 V:

```
rawReading ≈ 4095
```

Expected:

```
shockTravel = 0 mm
```

---

## TestCase 3 (Mid Travel)

Voltage ≈ **4.93 V** (half of 0.14 V span)

Expected:

```
shockTravel ≈ 0.5 × SHOCK_TRAVEL_MAX_MM
```

---

## TestCase 4 (Full Compression)

Voltage ≈ **4.86 V**

Expected:

```
shockTravel = SHOCK_TRAVEL_MAX_MM
```

---

## TestCase 5 (Out of Range)

Voltage difference exceeds **0.14 V**.

Expected:

```
constrain() clamps value to SHOCK_TRAVEL_MAX_MM
```
