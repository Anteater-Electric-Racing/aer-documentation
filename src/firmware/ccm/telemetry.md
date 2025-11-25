# Telemetry
Current Version: HIMaC testing:
```
telemetryData = {
            .APPS_Travel = APPS_GetAPPSReading(),

            .motorSpeed = MCU_GetMCU1Data()->motorSpeed,
            .motorTorque = MCU_GetMCU1Data()->motorTorque,
            .maxMotorTorque = MCU_GetMCU1Data()->maxMotorTorque,
            .motorState = Motor_GetState(),
            .mcuMainState = MCU_GetMCU1Data()->mcuMainState,
            .mcuWorkMode = MCU_GetMCU1Data()->mcuWorkMode,

            .mcuVoltage = MCU_GetMCU3Data()->mcuVoltage,
            .mcuCurrent = MCU_GetMCU3Data()->mcuCurrent,
            .motorTemp = MCU_GetMCU2Data()->motorTemp,
            .mcuTemp = MCU_GetMCU2Data()->mcuTemp,

            .dcMainWireOverVoltFault =
                MCU_GetMCU2Data()->dcMainWireOverVoltFault,
            .dcMainWireOverCurrFault =
                MCU_GetMCU2Data()->dcMainWireOverCurrFault,
            .motorOverSpdFault = MCU_GetMCU2Data()->motorOverSpdFault,
            .motorPhaseCurrFault = MCU_GetMCU2Data()->motorPhaseCurrFault,

            .motorStallFault = MCU_GetMCU2Data()->motorStallFault,
            .mcuWarningLevel = MCU_GetMCU2Data()->mcuWarningLevel,

        };
```

This telemetry section acts as a real-time summary of everything important happening in the drivetrain during HIMaC testing. It records the driver’s accelerator pedal position so we know how much torque is being requested at any given moment. Keeping this information in one place makes it easy to log, monitor, and analyze how the system responds under different conditions.

It also gathers detailed status information from the motor controller, including actual motor speed, delivered torque, voltage, current, and temperatures. These values help confirm that the system is performing as expected and staying within safe operating limits. Along with this, the telemetry captures the current operating modes and internal controller states, giving a clear picture of what the motor system is actively doing.

In addition, the telemetry includes a set of fault and warning indicators that report when something abnormal has occurred, such as overcurrent, overspeed, overheating, or voltage problems. These alerts allow the system to react quickly by limiting torque or switching into a safer mode. Overall, this telemetry structure provides a clean and organized snapshot of the drivetrain’s condition, making testing, troubleshooting, and performance analysis much easier.

All this data is sent over CAN to our Raspberry PI in 1 packet.
