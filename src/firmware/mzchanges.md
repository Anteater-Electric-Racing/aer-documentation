Author: Karan Thakkar
# KZ --> MZ Updates
MZ is our 2026 competition spec car. It is an **iteration** of KZ, and as such you will find most of the firmware to be very similar.

**This decision was made to minimize scope change and push UCI to bring their first vehicle to the annual FSAE competition.**

## Summary of Changes

- Reorganized file structures
- Firmware is split into:
```
src/
    peripherals                    // handles low level drivers and wrappers for communication protocols
    utils                           // useful constants
    vehicle/
        comms                       //communication handler over CAN
        controls                    //custom algorithms designed for specific use cases
        devices                     //logic level interface with peripherals and vehicle state
```
- New Inverter: DTI HV550, new motor: EMRAX 228 MV
    - The logic stays the same but the handler changes

- New features include:
    - Watchdog Timer
    - Sensors Added (Wheel speed, Shock travel)
    - Regen Braking Map
    - Migrate Traction Control, Launch Control from KZ
    - TSSI bypass per FSAE rules
    - PCC thermistors data + Plug and Play Charging
    - System derating based on temps
    - Closed loop control of Fans/Pumps


## Moving forward into Fall 2026
The goal is to create our own dev board, write more bare-metal code, and build the backend for reliable autonomy.

You can expect the firmware team to have minimal tasks as we transition to autonomy-centric but rather be a team that maintains
a reliable system as hardware changes.
