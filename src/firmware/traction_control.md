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

## Current Limitations
 
- PID gains (`TC_KP`, `TC_KI`, `TC_KD`) are placeholder values and require tuning.
- The target slip ratio (0.10) might require tweaking for effective traction control.
- Testing is yet to be done to ensure everything works well.
 