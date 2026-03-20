# Onboarding

Author: Karan Thakkar

## Relevant coding background to understand firmware
Git: [Video](https://www.youtube.com/watch?v=TFhbv6gw2Wo) || [Article](https://dev.to/ajmal_hasan/beginner-friendly-git-workflow-for-developers-2g3g)

C++ reference and language: [cppreference.com](https://en.cppreference.com/)
PlatformIO: [platformio.org](https://platformio.org/) — helpful for faster prototyping and device management.

FreeRTOS official site and API reference: [freertos.org](https://www.freertos.org/)

Debugging & diagnostics
- TBD (openCD?)

## Relevant hardware background to understand firmware
Teensy 4.1: [PJRC Teensy 4.1](https://www.pjrc.com/teensy/teensy41.html) - MCU board used in our stack

CAN protocol: [Video](https://www.youtube.com/watch?v=JZSCzRT9TTo)

## Custom PCBs
#### [CCM](https://github.com/Anteater-Electric-Racing/Central-Computer-Module)
Controls all the sensor inputs + outputs and perform motor control calculations.
#### [PCC](https://github.com/Anteater-Electric-Racing/Precharge-Circuit)
Automates the battery precharge sequence to mimize instaneous capacitor charging.
## Codebase overview
Here is our codebase: [https://github.com/Anteater-Electric-Racing/embedded](https://github.com/Anteater-Electric-Racing/embedded)
### [fsae-vehicle-fw](https://github.com/Anteater-Electric-Racing/embedded/tree/main/fsae-vehicle-fw)
Contains all vehicle firmware for CCM (sensor interfacing + motor controls)
### [fsae-pcc](https://github.com/Anteater-Electric-Racing/embedded/tree/main/fsae-pcc)
Contains all firmware for PCC sequence.
