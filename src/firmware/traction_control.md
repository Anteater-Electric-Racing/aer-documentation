# Traction Control Module 
 
Author: Atharva Rao
 
## Purpose
 
The traction control module prevents rear-wheel spin during acceleration by reducing the torque request sent to the inverter. It compares front (undriven) wheel speeds to the motor-derived rear wheel speed to detect slip, then applies a PID correction when slip exceeds a defined ratio.
 
---
 
## Control Method
 
### Slip Calculation
 
Slip is computed in `VCU_GetSlip()` using motor eRPM as the driven wheel speed reference, which is calculated through pole pairs and gear ratio:
 
```
motor_speed = (eRPM / POLE_PAIRS) / GEAR_RATIO
slip = (motor_speed − front_wheel_avg) / front_wheel_avg
```
 
The calculation also consists of an additional step: if the two front wheel speeds differ by more than `MAX_SPEED_DIFF` (35 RPM), the lower reading is used instead of the average to guard against a faulty sensor pulling the wheel speed reference high.
 
If the front wheel average falls below `MIN_SPEED` (5 RPM) — i.e., the car is nearly stationary, the function returns the target slip ratio directly, avoiding a division-by-near-zero.
 
### PID Torque Reduction
 
The PID controller targets `SLIP_RATIO` (0.10) as its setpoint. Its output is a multiplicative reduction applied to the driver's requested torque:
 
```
target = requested_torque − (requested_torque × PID_output)
```
This means that the correction scales with the driver's pedal demand. For example, a large torque request gets a larger absolute cut when the calculated slip is high.

Note: The PID output is constrained so the torque cannot be increased beyond what the driver requested (This is implemented to abide by the rules).


## Test Cases
 
Constants assumed: `POLE_PAIRS = 10`, `GEAR_RATIO = 3`, `MIN_SPEED = 5 RPM`, `MAX_SPEED_DIFF = 35 RPM`, `SLIP_RATIO = 0.10`, `TC_KP = 0.5`, `TC_KI = 0.01`, `TC_KD = 0.0`
 
---
 
## Test Case 1 — No Slip 
 
WSS1 = 100 RPM, WSS2 = 100 RPM, eRPM = 300000
 
requested_torque = 200 Nm
 
slip = (100 − 100) / 100 = 0.00
 
PID output = 0.5 × (0.00 − 0.10) = −0.05 → clamped to 0
 
torqueDemand = 200 Nm (unchanged)
 
---
 
## Test Case 2 — Slip At Threshold
 
WSS1 = 100 RPM, WSS2 = 100 RPM, eRPM = 330000
 
requested_torque = 200 Nm
 
motor_speed = (330000 / 10) / 3 = 110 RPM
 
slip = (110 − 100) / 100 = 0.10
 
PID output = 0.5 × (0.10 − 0.10) = 0.00
 
torqueDemand = 200 Nm (unchanged)
 
---
 
## Test Case 3 — Moderate Slip
 
WSS1 = 100 RPM, WSS2 = 100 RPM, eRPM = 360000
 
requested_torque = 200 Nm
 
motor_speed = (360000 / 10) / 3 = 120 RPM
 
slip = (120 − 100) / 100 = 0.20
 
PID output = 0.5 × (0.20 − 0.10) = 0.05
 
torqueDemand = 200 − (200 × 0.05) = 190 Nm
 
---
 
## Test Case 4 — Heavy Slip
 
WSS1 = 40 RPM, WSS2 = 42 RPM, eRPM = 300000
 
requested_torque = 200 Nm
 
motor_speed = (300000 / 10) / 3 = 100 RPM, front_avg = 41 RPM
 
slip = (100 − 41) / 41 = 1.44
 
PID output = 0.5 × (1.44 − 0.10) = 0.67
 
torqueDemand = 200 − (200 × 0.67) = 66 Nm
 

## Current Limitations
 
- PID gains (`TC_KP`, `TC_KI`, `TC_KD`) are placeholder values and require tuning.
- The target slip ratio (0.10) might require tweaking for effective traction control.
- Testing is yet to be done to ensure everything works well.
 