# Godot Game Engine Simulation

Author: Lawrence Chan

### Overview

Virtual simulation utilizing Godot Game Engine for testing vehicle characteristics / autonomous driving.

---

# Vehicle Controller Script (Godot – VehicleBody3D)

## Overview

This script implements a basic vehicle controller for a **Godot `VehicleBody3D` node**.

It handles:

* Acceleration and engine force
* Braking
* Steering control
* Natural deceleration
* Speed limiting
* Debug speed output

The script reads player inputs and updates the physics behavior of the vehicle's `VehicleWheel3D` nodes accordingly.

---

# Node Requirements

The script must be attached to a **VehicleBody3D** node.

The vehicle should contain child nodes of type:

* `VehicleWheel3D` with `use_as_traction = true` (drive wheels)
* `VehicleWheel3D` with `use_as_steering = true` (steering wheels)

---

# Input Controls

| Key | Action                   |
| --- | ------------------------ |
| W   | Accelerate               |
| A   | Turn left                |
| S   | Brake                    |
| D   | Turn right               |
| C   | Reset steering to center |

These must be mapped in **Project Settings → Input Map**:

* `accelerate`
* `decelerate`
* `steer_left`
* `steer_right`
* `steer_center`

---

# Configuration Constants

## Simulation Constants (Latest)

| Constant                | Current Value | Unit               | Function                                                                                    | Effect When Increased                |
| ----------------------- | ------------- | ------------------ | ------------------------------------------------------------------------------------------- | ------------------------------------ |
| `MAX_POWER`             | 500.0         | engine force (N)       | Maximum engine force applied to traction wheels                                             | Stronger acceleration                |
| `ACCELERATION_STEP`     | 5.0           | engine force/frame (N/frame) | Increment added to engine force each frame when accelerating (if dynamic throttle disabled) | Faster throttle response             |
| `USE_DYNAMIC_THROTTLE`  | true          | boolean            | Enables throttle control using `throttle_function()` instead of incremental acceleration    | Allows realistic engine power curves |
| `MOTOR_RAMP_DOWN_STEP`  | 50.0          | engine force/frame (N/frame) | Rate at which engine force decreases when accelerator is released                           | Vehicle loses power faster           |
| `BRAKING_STEP`          | 1.0           | brake force/frame  | Increment added to braking force when braking                                               | Brakes apply faster                  |
| `BASE_DECEL`            | 5.0           | brake force (N)        | Passive braking applied when no throttle or brake input is given                            | Vehicle slows faster when coasting   |
| `MAX_BRAKING`           | 15.0          | brake force (N)        | Maximum braking force that can be applied                                                   | Shorter stopping distance            |
| `STEER_STEP`            | 0.5           | degrees/frame      | Amount steering angle changes each frame when turning                                       | Faster steering response             |
| `MAX_STEERING_ANGLE`    | 32.0          | degrees            | Maximum allowed steering angle for steering wheels                                          | Tighter turning radius               |
| `MAX_VEHICLE_SPEED_MPH` | 90.0          | mph                | Maximum allowed vehicle speed before throttle stops increasing                              | Higher top speed                     |
| `DISPLAY_DEBUG_SPEED`   | true          | boolean            | Enables console output of speed and steering data                                           | Debug output enabled                 |


## Debug

```gdscript
const DISPLAY_DEBUG_SPEED = true
```

Prints vehicle speed and steering data to the console.

---

## Engine / Acceleration

| Constant             | Description                                         |
| -------------------- | --------------------------------------------------- |
| MAX_POWER            | Maximum engine force                                |
| ACCELERATION_STEP    | Increment applied to engine force when accelerating |
| USE_DYNAMIC_THROTTLE | Enables dynamic throttle based on speed             |
| MOTOR_RAMP_DOWN_STEP | Rate engine force decreases when not accelerating   |

---

## Braking

| Constant     | Description                                 |
| ------------ | ------------------------------------------- |
| BRAKING_STEP | Brake force increment                       |
| BASE_DECEL   | Passive braking when no throttle is applied |
| MAX_BRAKING  | Maximum brake force                         |

---

## Steering

| Constant           | Description                             |
| ------------------ | --------------------------------------- |
| STEER_STEP         | Steering change per frame               |
| MAX_STEERING_ANGLE | Maximum wheel steering angle in degrees |

---

## Speed Limit

```gdscript
const MAX_VEHICLE_SPEED_MPH = 90.0
```

Maximum allowed vehicle speed.

Internally the script uses **m/s and km/h**, converting to mph only for debug display.

---

# Core Variables

| Variable                 | Purpose                                 |
| ------------------------ | --------------------------------------- |
| previous_displayed_speed | Prevents duplicate debug prints         |
| steer_normalized         | Normalized steering value for debugging |

---

# Utility Functions

## Degree / Radian Conversion

```gdscript
func deg_to_rad(deg):
    return deg * PI / 180

func rad_to_deg(rad):
    return rad * 180 / PI
```

Used for steering calculations.

---

## Speed Conversion

```gdscript
func kph_to_mph(kph) -> float:
    return kph / 1.609
```

Converts kilometers per hour to miles per hour.

---

## Throttle Function

```gdscript
func throttle_function(current_speed) -> float:
    # Connect engine power data here
    return 1000.0
```

Returns engine force when **dynamic throttle** is enabled.

This is currently a placeholder and can be replaced with a realistic engine torque curve.

---

## Normalized Steering Angle

```gdscript
func norm_steering_angle(steer: float) -> float:
    return steer * (90/PI)
```

Converts steering radians into a normalized value for debugging.

---

# Physics Update Loop

Main vehicle logic runs inside:

```gdscript
func _physics_process(delta: float) -> void:
```

This function performs:

1. Input processing
2. Debug speed calculation
3. Wheel physics updates

---

# Speed Calculation

Vehicle speed is calculated from the body's velocity:

```gdscript
var velocity_mps = linear_velocity.length()
```

Conversions:

* m/s → km/h: `mps * 3.6`
* km/h → mph: `kph / 1.609`

Example debug output:

```
VEHICLE SPEED | 72 km/h [20 m/s] (45 mph) | STEER: 0.4
```

---

# Wheel Processing

The script loops through all child nodes:

```gdscript
for object in get_children():
```

If the node is a `VehicleWheel3D`, it is processed depending on its role.

---

# Traction Wheels

If:

```gdscript
wheel.use_as_traction
```

The script applies engine and braking forces.

## Acceleration

Engine force increases while:

* Below `MAX_POWER`
* Below `MAX_VEHICLE_SPEED`

## Braking

Brake force increases up to:

```
MAX_BRAKING
```

Engine force is set to zero when braking.

## Natural Deceleration

When neither accelerating nor braking:

* Engine force gradually decreases
* Base braking (`BASE_DECEL`) is applied

---

# Steering Wheels

If:

```gdscript
wheel.use_as_steering
```

The script adjusts the wheel steering angle.

Steering changes are converted from degrees to radians because Godot steering uses radians.

---

# Steering Limits

Maximum steering range:

```
± MAX_STEERING_ANGLE
```

If exceeded, the value is clamped.

---

# Steering Reset

Pressing **C** resets steering:

```gdscript
wheel.steering = 0.0
```

---

# Vehicle Behavior Summary

| Behavior         | Description                                   |
| ---------------- | --------------------------------------------- |
| Acceleration     | Applies engine force to traction wheels       |
| Braking          | Applies brake force and disables engine force |
| Steering         | Rotates steering wheels within limits         |
| Natural slowdown | Engine force gradually decreases              |
| Speed limiting   | Prevents vehicle exceeding maximum speed      |
| Debug output     | Displays speed and steering data              |

---

# Possible Improvements

* Implement a realistic **engine torque curve**
* Add **gear ratios and transmission**
* Implement **traction control**
* Add **drift or slip simulation**
* Display speed using a **UI element instead of console output**

---

# Example Node Structure

```
VehicleBody3D
│
├── VehicleWheel3D (FrontLeft)
├── VehicleWheel3D (FrontRight)
├── VehicleWheel3D (RearLeft)
└── VehicleWheel3D (RearRight)
```

Typical configuration:

| Wheel       | Traction | Steering |
| ----------- | -------- | -------- |
| Front Left  | No       | Yes      |
| Front Right | No       | Yes      |
| Rear Left   | Yes      | No       |
| Rear Right  | Yes      | No       |

---

# Summary

This script provides a **basic but functional vehicle physics controller** using Godot's built-in vehicle system.

It demonstrates:

* Player input handling
* Vehicle physics updates
* Wheel-based acceleration and steering
* Debug speed monitoring

The system is designed to be **simple, modular, and easily extendable** for more advanced driving simulations.
