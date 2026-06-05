# Regenerative Braking

**Anoop Koganti**

---

# Goal

Design a mapped braking system that properly integrates motor braking for optimal regenerative braking while providing a validated and tuned transition from motor braking to mechanical braking.

---

# High-Level Overview

## System Flow

```text
Brake System Encoder
        ↓
      CCM
        ↓
Motor Regen Command
        ↓
Motor Braking (0-20% Encoder Travel)
        ↓
Mechanical Braking (>20% Encoder Travel)
        ↓
Brake Pressure Generated Through Excess Slack
```

### Concept

- The brake encoder is used as the primary input.
- The CCM reads the encoder value and determines the requested braking level.
- The first 20% of calibrated brake travel is dedicated to regenerative braking.
- Regen torque is linearly mapped from 0% to 100% over this range.
- After 20% brake travel, regen remains at its maximum value.
- Additional brake pedal travel engages the mechanical braking system through intentional slack in the brake linkage.
- Mechanical brake pressure must be tuned to account for the additional braking force produced by the motor.

---

# Update Function

```c
int BrakesRegen(int EncoderADC);
```

## Function Purpose

- Takes brake encoder ADC readings as input.
- Maps encoder position from the calibrated 0-20% brake travel range.
- Outputs a regenerative braking request from 0-100%.
- Sends the corresponding negative torque request to the motor controller.

## Example Mapping

| Encoder Travel | Regen Output |
|---------------|-------------|
| 0% | 0% |
| 5% | 25% |
| 10% | 50% |
| 15% | 75% |
| 20% | 100% |
| >20% | 100% |

---

# Mechanical Braking Considerations

## Brake Bias Tuning

The rear motor contributes braking torque during regenerative braking. Mechanical brake pressure must therefore be tuned such that total vehicle braking remains balanced.

Areas requiring validation:

- Front-to-rear brake balance
- Vehicle stability during braking
- Smooth transition from regen-only braking to regen + mechanical braking
- Driver pedal feel

## Calibration Approach

- Begin with conservative regen values.
- Incrementally increase regen contribution.
- Record stopping behavior and vehicle response.
- Adjust brake pressure and slack distance as necessary.
- Validate consistency across multiple runs.

---

# Initial Test Cases

## Test 1: Encoder Range Mapping

### Input

Encoder values corresponding to:

- 0%
- 5%
- 10%
- 15%
- 20%

### Expected Result

Regen command increases linearly from 0% to 100%.

### Verification

Confirm motor controller receives the expected negative torque command.

---

## Test 2: Regen Saturation

### Input

Encoder values greater than 20% travel.

### Expected Result

Regen remains capped at 100%.

### Verification

Motor torque command does not increase beyond the calibrated maximum.

---

## Test 3: No Brake Input

### Input

0% brake travel.

### Expected Result

0% regenerative braking.

### Verification

No negative torque command is sent to the motor.

---

## Test 4: Mechanical Brake Transition

### Input

Brake pedal moved from below 20% to above 20%.

### Expected Result

- Regen reaches maximum at 20%.
- Mechanical brakes begin contributing after slack is removed.
- Braking force increases smoothly.

### Verification

No sudden jump in braking force.

---

## Test 5: Low-Speed Behavior

### Input

Brake applied while vehicle speed approaches zero.

### Expected Result

Regen is reduced or disabled at low speed.

### Verification

Vehicle comes to a smooth stop without oscillations or jerking.

---

## Test 6: BMS Fault Handling

### Input

BMS reports a charging fault or disables charging.

### Expected Result

- Regen command immediately becomes zero.
- Mechanical braking remains available.

### Verification

- No negative motor torque is requested.
- Vehicle can still brake mechanically.

---

## Test 7: Motor Controller Fault Handling

### Input

Motor controller fault condition.

### Expected Result

- Regen command immediately becomes zero.
- Mechanical braking remains available.

### Verification

- No regenerative torque request is sent.
- Vehicle can still brake mechanically.

---

## Test 8: Repeatability Test

### Input

Repeated brake applications at identical encoder positions.

### Expected Result

Same encoder value produces the same regen command each time.

### Verification

Mapping remains stable and repeatable.

---

# Future Work

- Determine optimal regen percentage range.
- Determine ideal slack distance before mechanical brake engagement.
- Tune brake bias for motor-assisted braking.
- Add battery current limiting.
- Add state-of-charge-based regen limiting.
- Add thermal derating based on motor and battery temperature.
- Collect braking data for validation and tuning.
