Author: Karan Thakkar
# Peripherals
## Digital I/O

### RTM (Ready To Move) Button Input
The RTM input is handled as a standard digital GPIO read. It functions as a momentary push-button with software debouncing to avoid false triggering from mechanical bounce. Each press toggles the latched internal state between 0 and 1, allowing the system to reliably detect transitions. This signal is used in the vehicle state machine to move the car from the IDLE state into the RUNNING or DRIVE state once all safety conditions are met.

### Brake Light Output
The brake light is driven as a digital output that activates based on braking conditions. Depending on system rules and logic, this signal can be turned on when:
- Brake pressure exceeds a threshold
- Regenerative braking torque is applied above a defined amount
- Other system-defined braking conditions are met
The output is typically a high-side digital drive that activates the physical brake lamp in accordance with FSAE regulations.

### WheelSpeed Sensors (Front) Input
Front wheel speed sensors are hall-effect pickups producing a square-wave digital signal proportional to wheel rotation. The frequency of the pulses corresponds to wheel speed, allowing RPM to be computed from:
- The frequency of the input pulses
- The known number of sensor teeth or magnets
- Wheel circumference and drivetrain ratios

From this, speed and wheel rotational dynamics can be calculated for traction control, odometry, telemetry, and dynamic vehicle control.

### Fans & Pumps Output
Two MOSFET-controlled outputs are available for driving the cooling fan and coolant pump. These are controlled through software feedback loops that monitor thermistor temperature readings. When temperature increases beyond a target setpoint, fan or pump duty cycle increases proportionally. When temperatures fall below defined limits, duty cycle is reduced. This provides automatic thermal regulation for the accumulator or power electronics cooling loop.

### Speaker Output
Audio signaling is performed using a PWM output connected to an amplifier which drives the speaker. This is used to produce audible alerts per competition requirements, such as:
- Ready-to-drive tone
- Fault indication tones
- System status alerts

The speaker system ensures the car meets FSAE safety requirements for audible signaling before the drivetrain engages.

---

## Analog I/O

### Brake System Encoder
A linear analog pressure transducer is used to measure hydraulic brake line pressure. The voltage is read via an ADC and mapped into engineering units using a calibration curve. Brake pressure is used to:
- Smoothly blend mechanical and regenerative braking
- Block acceleration torque while braking
- Activate brake lights
- Support telemetry and control algorithms

This sensor is part of multiple safety-critical paths and must be sampled and checked reliably.

### Acceleration Pedal Position
The accelerator pedal position sensor outputs a variable voltage proportional to throttle demand. The reading is converted into a normalized torque request and fed into the torque controller. For safety, dual redundant channels are commonly monitored, and mismatches are checked to detect faults such as wiring damage or sensor failure. If readings disagree beyond limits, torque is disabled and a fault is raised.

### Suspension Travel
Suspension position is measured by analog linear potentiometers  These readings allow:
- Logging for vehicle dynamics analysis
- Dynamic control adjustments such as damping or traction control

### Thermistors
Thermistors measure temperature in critical subsystems such as:
- Motor controllers
- Batteries and accumulator modules
- Cooling loops
- Power electronics

These readings are used in closed-loop thermal regulation. Raw voltage readings are converted using standard thermistor curve equations or lookup tables. If temperatures exceed defined safety thresholds, torque may be reduced, cooling activated, or the vehicle shut down depending on severity.

---

## CAN

### BMS
The Battery Management System communicates via CAN, sending essential data such as:
- Cell voltages
- Module temperatures
- Current consumption
- State-of-Charge and State-of-Health
- Fault flags and system warnings

The control system must consume these messages to maintain safe vehicle operation. If BMS signals a critical fault, the drivetrain must disable torque output immediately.

### Battery Charger
During charging mode, the onboard charger exchanges CAN messages with the BMS and vehicle controller. Messages include:
- Charging current and voltage targets
- Charge state progress
- Fault and error reporting

The charger generally follows BMS instructions to ensure safe constant-current and constant-voltage charging curves.

### PCC
The PCC (Pre Charge Circuit) communicates over CAN to exchange infor with the CCM. Data typically includes:
- Precharge state
- Precharge Progress
- TS/Accum Voltage read over frequency to voltage circuits
- Precharge time

The PCC and CCM coordinate to ensure safe charing of the inverter and HV side. Read more about this in the PCC section.

### CCM
The CCM (Central Computer Module) acts as the main MCU for car.
- Broadcast system heartbeat messages
- Reads all sensor readings
- Receive driver input and button states
- Relay fault conditions
- Coordinate operation between subsystems
- Sends torque commands to the Inverter based on all readings
- Relays info to telemetry side.
