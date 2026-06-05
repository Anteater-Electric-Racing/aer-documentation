 <div align="center">

   # Launch Control Development Plan
   #### Anoop Koganti, Rishi Theegala, Pranav Rayapureddi
</div>


## Purpose
Design a function that optimizes **0–60 mph acceleration** by adjusting torque output based on **slip ratio**.
- Optimal slip ratio ≈ **7%** (commonly 5–10% depending on car).
- If slip ratio **exceeds** optimal → **decrease torque**  
- If slip ratio **falls below** optimal → **increase torque**

---

## Algorithm Overview
### Initial Function
- Validate and reset PID gains and internal PID state.
- Obtain constants (max torque, optimal slip target).
- Initialize launch control button to false (inactive until pressed).

### Pre-Launch Phase
- Open loop torque ramp over 500ms, capped at 60% of `CAPPED_MOTOR_TORQUE`
- Transitions to Post-Launch when `freeSpeed > 0.1 mph`, resets PID on transition
- If Pre-Launch exceeds 2 seconds → exit to OPEN_LOOP

---

### Post-Launch Phase
1. **Check button state**
   - If not pressed → output driver torque normally.
2. **Wheel speeds**
   - Controlled speed: **rear wheel** with **higher** speed.
   - Free rolling speed: **front wheel** with **lower** speed.
3. **Slip Ratio**
   - slip = (controlledSpeed - freeRollingSpeed) / freeRollingSpeed
4. **Torque Logic**
- If slip < 0.07 → increase torque.  
- If slip > 0.07 → decrease torque.
5. **PID Correction**
- Use `PID::computePID()` for correction.
- Add correction to driver torque.
6. **Clamp Torque**
- If corrected torque > max → clamp to maxTorque.
- If < 0 → clamp to 0 Nm.
---
## Implementation Details

**LAUNCH_STATE_ON**
**PRE_LAUNCH (0 → 0.1 mph)**
1. Get front wheel speeds (MPH), take slower as `freeSpeed`
2. Check if elapsed time > 2s → exit to OPEN_LOOP
3. If `freeSpeed > 0.1 mph` → transition to POST_LAUNCH, reset PID
4. Ramp torque linearly over 500ms, capped at 60% of `CAPPED_MOTOR_TORQUE`

**POST_LAUNCH (0.1 mph → launchTargetSpeed)**
1. Compute `controlledSpeed` from motor eRPM × rpmConversion × wheelRadius
2. Compute slip ratio = (controlledSpeed - freeSpeed) / freeSpeed
3. PID outputs correction, add to normalized `realTorque`
4. Multiply by `CAPPED_MOTOR_TORQUE`
5. Clamp target between `minTorque` and `CAPPED_MOTOR_TORQUE`
6. If `freeSpeed > launchTargetSpeed` → exit to OPEN_LOOP, reset PID.

**LAUNCH_STATE_OFF**
- Wait for:
- Motor speed = 0
- Launch button activated
- Then activate launch control (FSAE requires button activation — must update logic).

**FAULT State**
- Triggered by irregular wheel speeds or invalid conditions.

---

# TESTING PLAN

Constants:

    slipTarget = 0.7f

    minTorque = 0.0f

    maxTorque = 260.0f

    wheelRadius = 0.35f

    rpmConversion = 0.3f

    INTEGRAL_MAX = 20.0f

    INTEGRAL_MIN = -20.0f

    launchTargetSpeed = 10.0f
<div align="center">

## TestCase 1 (Entry Condition)
eRPM = 0.0
pedalAccel = 1.0 (fully pressed)
driveStrategy = OPEN_LOOP
Expected: launchState → PRE_LAUNCH, driveStrategy → LAUNCH_CTRL, PID reset

</div>
<div align="center">

## TestCase 2 (PRE_LAUNCH Ramp)
timeElapsed = 0.25s (halfway through ramp)
freeSpeed = 0.0 mph
Expected: target = 0.3 × CAPPED_MOTOR_TORQUE (30%, halfway to 60% cap)
launchState = PRE_LAUNCH

</div>
<div align="center">

## TestCase 3 (PRE_LAUNCH → POST_LAUNCH Transition)
freeSpeed = 0.15 mph (crosses 0.1 threshold)
Expected: launchState → POST_LAUNCH, PID reset

</div>
<div align="center">

## TestCase 4 (POST_LAUNCH high slip)
wheelSpeedFL = 40 mph, wheelSpeedFR = 42 mph, controlledSpeed = 100 mph
realTorque = 200 Nm (normalized: 0.77)
slipRatio = (100 - 40) / 40 = 1.5 → very high slip
PID correction → negative
target reduced toward minTorque = 0 Nm

</div>
<div align="center">

## TestCase 5 (POST_LAUNCH low slip)
wheelSpeedFL = 100 mph, wheelSpeedFR = 99 mph, controlledSpeed = 100 mph
realTorque = 250 Nm (normalized: 0.96), slip slightly below 0.07
PID correction → positive
target → clamped at CAPPED_MOTOR_TORQUE = 260 Nm

</div>
<div align="center">

## TestCase 6 (PRE_LAUNCH timeout)
timeElapsed = 2.1s, freeSpeed = 0.0 mph (car never moved)
Expected: driveStrategy → OPEN_LOOP, launchState → PRE_LAUNCH

</div>
<div align="center">

## TestCase 7 (Exit — throttle lifted)
driveStrategy = LAUNCH_CTRL
APPS_GetAPPSReading() = 0.03 (below 0.05 threshold)
Expected: driveStrategy → OPEN_LOOP, PID reset

</div>
<div align="center">

## TestCase 8 (Exit — target speed reached)
freeSpeed = 11.0 mph (above launchTargetSpeed = 10.0)
Expected: driveStrategy → OPEN_LOOP, launchState → PRE_LAUNCH, PID reset

</div>
