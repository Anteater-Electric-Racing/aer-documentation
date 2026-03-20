# Traction Control Module Overview

This module implements a basic traction‑control strategy intended to limit excessive wheel slip during acceleration. The approach uses front‑wheel speeds as an estimate of vehicle speed and compares them to the driven rear‑wheel speeds to detect slip. When slip exceeds a defined threshold, the system reduces the requested torque using a PID controller.

### Purpose  
The goal of this traction‑control logic is to prevent rear‑wheel spin by dynamically adjusting torque output. By monitoring the difference between front and rear wheel speeds, the system attempts to maintain the driven wheels within an acceptable slip range, improving vehicle stability and acceleration performance.

### Control Method  
The algorithm calculates slip using the ratio:

slip = (rear wheel speed average - front wheel speed average)/(front wheel speed average)

A PID controller is configured to reduce torque when slip exceeds a predefined threshold. The PID output directly modifies the torque request sent to the powertrain, with proportional control currently being the primary active term.

### System Behavior  
- Front wheel speeds serve as a reference for true vehicle speed.  
- Rear wheel speeds represent the driven wheels, where slip is expected to occur.  
- When slip rises above the threshold, the PID controller computes a corrective torque reduction.  
- The loop continuously monitors wheel speeds and updates torque in real time.

### Current Limitations  
This implementation is an early prototype and is not yet suitable for on‑vehicle use. Key limitations include:  
- The PID controller is re‑initialized on every loop iteration, preventing proper integral and derivative behavior.  
- The control loop runs indefinitely without timing control, blocking other system tasks and preventing consistent update rates.  
- The slip threshold is set unrealistically high for effective traction control.  
- Wheel‑speed inputs are unfiltered, making the system sensitive to noise.  
- No safety checks or torque‑limiting constraints are implemented beyond the raw PID output.
