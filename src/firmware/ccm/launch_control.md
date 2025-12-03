 <div align="center">

   # Launch Control Development Plan
   #### Anoop Koganti, Rishi Theegala
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

---

### Update Function

1. **Check button state**
   - If not pressed → output driver torque normally.

2. **Wheel speeds**
   - Controlled speed: **rear wheel** with **higher** speed.
   - Free rolling speed: **front wheel** with **lower** speed.

3. **Slip Ratio**
   - slip = (controlledSpeed - freeRollingSpeed) / freeRollingSpeed

4. **Torque Logic**
- If slip < 0.05 → increase torque.  
- If slip > 0.10 → decrease torque.

5. **PID Correction**
- Run at 100 Hz (`dt = 0.01`).
- Use `PID::PID_Calculate()` for correction.
- Add correction to driver torque.

6. **Clamp Torque**
- If corrected torque > max → clamp to maxTorque.
- If < 0 → clamp to 0 Nm.

---

## Implementation Details

### `void launchControl_Init()`
Initializes PID controller values for slip ratio correction.

---

### `void threadLaunchControl(void *pvParameters)`

**LAUNCH_STATE_ON**
1. Obtain slower front wheel speed.  
2. Convert rad/s → m/s using wheel radius.  
3. Motor/rear speed used as controlled speed.  
4. Compute slip.  
5. PID applies torque correction.  
6. Clamp torque within bounds.  
7. If brakes are pressed → exit to LAUNCH_STATE_OFF.

**LAUNCH_STATE_OFF**
- Wait for:
- Front brake fully pressed  
- Motor speed = 0  
- Then activate launch control (FSAE requires button activation — must update logic).

**FAULT State**
- Triggered by irregular wheel speeds or invalid conditions.

---

# TESTING PLAN

Method Signature: 

    threadLaunchControl(void *pvParameters)



Constants:

    slipTarget = 0.7f

    minTorque = 0.0f

    maxTorque = 260.0f

    wheelRadius = 0.35f



<div align="center">

## TestCase 1 (base case)

Brake = 60 psi

Motor speed = 0 rad/s

Motor state = MOTOR_STATE_DRIVING



threadLaunchControl()

launchControlState -> LAUNCH_STATE_ON

</div>



<div align="center">

## TestCase 2

wheelSpeedFL = 80 rad/s, wheelSPEEDFR = 82 rad/s, motorSpeed = 80 rad/s

realTorque = 150 Nm

LaunchState = on



Slip ratio = 0.0–0.02 -> small PID correction

torqueDemand = 150 Nm (unchanged to barely changed)

LaunchState = ON

</div>



<div align="center">

## TestCase 3

wheelSpeedFL = 40 rad/s, wheelSpeedFR = 42 rad/s, motorSpeed = 100 rad/s

realTorque = 200 Nm



Slip ratio = (100–41)/41 = 1.43 -> very high slip

PID correction negative -> torque reduced near 0 Nm

torqueDemand = near minTorque = around 0 Nm

</div>



<div align="center">

## TestCase 4

wheelSpeedFL = 100, wheelSpeedFR = 99, motorSpeed = 100

realTorque = 250 Nm, slip slightly below 0.07



PID positive correction increases torque above 260 -> clamped at 260 Nm

torqueDemand = 260 Nm

</div>



<div align="center">

## TestCase 5

LaunchState = ON, BSE_GetBSEReading() -> 80 PSI

APPS_GetAPPSReading() < 1.0



Expected behavior:



PID reset

torqueDemand = rawTorqueInput

launchControlState = LAUNCH_STATE_OFF

</div>
