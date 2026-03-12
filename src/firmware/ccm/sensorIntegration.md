<div align="center">

# Shock Sensor Integration Plan
#### Anoop Koganti

</div>

## Purpose
Design a function that converts **4 shock sensor ADC readings** into usable **shock travel values in millimeters** for telemetry and suspension analysis.

The function accounts for the fact that the linear potentiometer signal is **inverted** in the current wiring/ADC interpretation:
- Sensor output starts near **5 V** when the shock is **not compressed**
- The measurable voltage swing during travel is only about **0.14 V**
- Because of that, the code flips the ADC reading and maps the small voltage change into a **0 to max travel** range

This allows the system to track suspension compression for all four corners of the car.

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

   - This is done because the sensor behavior is effectively reversed in the current setup.
   - A 12-bit ADC ranges from **0 to 4095**, so subtracting from 4095 flips the scale.
   - Higher original ADC values become lower processed values, and vice versa.

3. **Convert ADC counts to voltage**
   - Each flipped ADC value is passed through:

     ```cpp
     ADC_VALUE_TO_VOLTAGE(...)
     ```

   - This converts the processed ADC count into a voltage value.

4. **Use `abs()` on voltage**
   - The code applies `abs()` to the converted voltage as a safety step so the stored voltage remains non-negative.
   - In practice, voltage should already be non-negative after conversion, so this mostly protects against unexpected sign behavior.

5. **Map voltage to shock travel**
   - Travel is calculated using:

     ```cpp
     (voltage / 0.14f) * SHOCK_TRAVEL_MAX_MM
     ```

   - Since the useful signal span is only **0.14 V**, the code scales that small range to the full mechanical travel.
   - So:
     - `0.00 V` corresponds to `0 mm`
     - `0.14 V` corresponds to `SHOCK_TRAVEL_MAX_MM`

6. **Clamp output**
   - Each travel value is constrained using:

     ```cpp
     constrain(value, 0.0f, SHOCK_TRAVEL_MAX_MM)
     ```

   - This prevents bad readings from going negative or exceeding the expected maximum travel.

---

## Why `abs()` is used

### 1. Flipping the ADC scale
The most important use of `abs()` is here:

```cpp
abs(4095 - rawReading)
```

Because the sensor sits near **5 V at full extension / no compression**, the ADC reading begins near the top of the ADC range. As the shock moves, the voltage only changes by about **0.14 V**, and in the current setup that change goes in the opposite direction of the desired travel interpretation.

So instead of directly mapping the raw ADC reading, the code mirrors it around the ADC maximum:
- near 4095 becomes near 0
- lower readings become larger processed values

That makes the travel calculation easier, because now increasing processed voltage corresponds more directly to increasing travel.

### 2. Keeping values positive
The voltage conversion also uses `abs()`:

```cpp
abs(ADC_VALUE_TO_VOLTAGE(...))
```

This keeps the voltage non-negative. It is more of a defensive cleanup step than the main calibration mechanism.

### Important note
`abs()` does **not** remove the 5 V offset by itself.

What really makes the mapping work is:
- flipping the ADC reading with `4095 - rawReading`
- then scaling the small changed portion of the signal by `0.14f`

So the logic is not “subtract 5 V directly.”  
Instead, it treats the **measured changing portion** of the signal as the useful range and stretches that range across the full shock travel.

---

## Implementation Details

### `void Shock_Init()`
Initializes all stored linear potentiometer data to zero.

This includes:
- raw ADC readings for all 4 shocks
- converted voltages for all 4 shocks
- shock travel values in mm for all 4 shocks

This function should be called once during system startup before sensor updates begin.

---

### `void ShockTravelUpdateData(uint16_t rawReading1, uint16_t rawReading2, uint16_t rawReading3, uint16_t rawReading4)`

Processes all 4 shock sensor readings and updates the global `linPots` structure.

#### Step-by-step behavior
1. Save processed raw readings after ADC inversion
2. Convert processed raw readings to voltages
3. Scale each voltage using the calibrated `0.14 V` span
4. Convert each scaled value into millimeters
5. Clamp each value from `0` to `SHOCK_TRAVEL_MAX_MM`

This function should be called whenever fresh ADC samples are available.

---

### `Linpot_GetData()`
Returns a pointer to the full `linPots` data structure so other modules can read:
- raw processed shock values
- voltages
- travel in mm

This is useful for telemetry, logging, suspension analysis, and future control features.

---

## Code Walkthrough

### Data storage
```cpp
static LinpotData linPots;
```

A single static struct stores all shock-related values in one place.

---

### Startup reset
```cpp
void Shock_Init() {
    linPots.LinPot1Voltage = 0;
    linPots.LinPot2Voltage = 0;
    linPots.LinPot3Voltage = 0;
    linPots.LinPot4Voltage = 0;

    linPots.shockTravel1_mm = 0;
    linPots.shockTravel2_mm = 0;
    linPots.shockTravel3_mm = 0;
    linPots.shockTravel4_mm = 0;

    linPots.Shock1RawReading = 0;
    linPots.Shock2RawReading = 0;
    linPots.Shock3RawReading = 0;
    linPots.Shock4RawReading = 0;
}
```

Everything is zeroed so the telemetry starts clean.

---

### ADC inversion
```cpp
linPots.Shock1RawReading = abs(4095 - rawReading1);
linPots.Shock2RawReading = abs(4095 - rawReading2);
linPots.Shock3RawReading = abs(4095 - rawReading3);
linPots.Shock4RawReading = abs(4095 - rawReading4);
```

This mirrors each 12-bit ADC reading around 4095.

---

### Voltage conversion
```cpp
linPots.LinPot1Voltage = abs(ADC_VALUE_TO_VOLTAGE(linPots.Shock1RawReading));
linPots.LinPot2Voltage = abs(ADC_VALUE_TO_VOLTAGE(linPots.Shock2RawReading));
linPots.LinPot3Voltage = abs(ADC_VALUE_TO_VOLTAGE(linPots.Shock3RawReading));
linPots.LinPot4Voltage = abs(ADC_VALUE_TO_VOLTAGE(linPots.Shock4RawReading));
```

After inversion, each reading is converted into voltage.

---

### Travel mapping
```cpp
linPots.shockTravel1_mm =
    constrain((linPots.LinPot1Voltage / 0.14f) * SHOCK_TRAVEL_MAX_MM, 0.0f,
              SHOCK_TRAVEL_MAX_MM);
```

The same mapping is repeated for all four sensors.

This line says:

- take measured voltage
- divide by the calibrated active voltage span `0.14f`
- multiply by max mechanical travel
- clamp into a valid range

Tiny voltage ripple, giant suspension meaning. Very bonsai signal, very full-size interpretation.

---

## How to Use

### 1. Initialize once
Call this at startup:

```cpp
Shock_Init();
```

---

### 2. Update with fresh ADC data
Whenever new ADC samples are read for the 4 shock sensors:

```cpp
ShockTravelUpdateData(raw1, raw2, raw3, raw4);
```

Where:
- `raw1` = shock 1 ADC reading
- `raw2` = shock 2 ADC reading
- `raw3` = shock 3 ADC reading
- `raw4` = shock 4 ADC reading

---

### 3. Read processed values
Use:

```cpp
LinpotData *data = Linpot_GetData();
```

Then access values like:
- `data->shockTravel1_mm`
- `data->shockTravel2_mm`
- `data->shockTravel3_mm`
- `data->shockTravel4_mm`

You can also access:
- processed raw values
- converted voltages

---

## Example

### Example input
Suppose:

```cpp
rawReading1 = 4095;
```

Then:

```cpp
abs(4095 - 4095) = 0
```

Voltage becomes about `0 V`, so travel becomes:

```cpp
(0 / 0.14) * SHOCK_TRAVEL_MAX_MM = 0 mm
```

Now suppose the sensor changes enough that the processed voltage becomes:

```cpp
0.07 V
```

Then travel becomes:

```cpp
(0.07 / 0.14) * SHOCK_TRAVEL_MAX_MM = 0.5 * SHOCK_TRAVEL_MAX_MM
```

So that corresponds to about **half of full travel**.

---

## Notes / Limitations

- The current mapping assumes the useful sensor range is exactly **0.14 V**
- If real measured sensor span differs corner-to-corner, each shock may need its own calibration
- Using `abs()` is helpful for inversion and safety, but true sensor calibration may eventually be better handled with:
  - measured minimum voltage
  - measured maximum voltage
  - per-sensor offsets
  - filtering for ADC noise

A more general future mapping could look like:

```cpp
travel = ((voltage - minVoltage) / (maxVoltage - minVoltage)) * SHOCK_TRAVEL_MAX_MM;
```

That would be cleaner if the team later performs a full calibration sweep.

---

# TESTING PLAN

Method Signature:

    ShockTravelUpdateData(uint16_t rawReading1, uint16_t rawReading2,
                          uint16_t rawReading3, uint16_t rawReading4)

Constants:

    ADC max count = 4095

    active voltage span = 0.14f

    minTravel = 0.0f

    maxTravel = SHOCK_TRAVEL_MAX_MM

<div align="center">

## TestCase 1 (initialization)

Call:

    Shock_Init()

Expected behavior:

- All raw readings = 0
- All voltages = 0
- All shock travel values = 0 mm

</div>

<div align="center">

## TestCase 2 (fully extended / no compression)

rawReading1 = 4095  
rawReading2 = 4095  
rawReading3 = 4095  
rawReading4 = 4095

Expected behavior:

- Processed raw values = 0
- Voltages = 0 V
- shockTravel1_mm through shockTravel4_mm = 0 mm

</div>

<div align="center">

## TestCase 3 (mid-travel equivalent)

Choose ADC values such that processed voltage is about 0.07 V

Expected behavior:

- Each affected shock reports about half of `SHOCK_TRAVEL_MAX_MM`

Example expectation:

    shockTravelX_mm ≈ 0.5 * SHOCK_TRAVEL_MAX_MM

</div>

<div align="center">

## TestCase 4 (full-scale travel)

Choose ADC values such that processed voltage is about 0.14 V

Expected behavior:

- Computed travel reaches max
- Output is clamped to:

    SHOCK_TRAVEL_MAX_MM

</div>

<div align="center">

## TestCase 5 (over-range input)

Choose ADC values that convert to a voltage greater than 0.14 V after inversion

Expected behavior:

- Travel calculation exceeds max internally
- `constrain()` clamps final value to:

    SHOCK_TRAVEL_MAX_MM

</div>

<div align="center">

## TestCase 6 (negative or invalid intermediate protection)

If any unexpected conversion behavior causes a negative intermediate value

Expected behavior:

- `abs()` forces non-negative voltage
- `constrain()` prevents negative travel
- Final output remains in valid range

</div>
