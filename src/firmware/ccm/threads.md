Author: Karan Thakkar
# Tasks & Scheduling

![" "](../../images/firmware/freertos.png)

The firmware is structured using FreeRTOS to split the vehicle control system into separate tasks, each handling its own area. ADC readings, motor control, telemetry, and high-level control all run in their own threads, so critical operations don’t get blocked by slower tasks. For example, threadADC continuously samples analog sensors, while threadMotor handles the motor state machine, updates torque, and communicates with the VCU and BMS over CAN. This keeps motor commands and sensor updates running on a predictable schedule, which is important for safe and reliable operation in the car.

The threadMain task is used for testing and user interaction. It reads input from the serial monitor to adjust torque, change vehicle states, and toggle regen, while also printing telemetry like motor speed, current, and temperatures. threadTelemetry collects and sends key vehicle data over CAN on a regular cycle. Separating everything into different threads makes the system easier to debug, maintain, and expand. FreeRTOS handles scheduling, so high-priority tasks like motor control always run reliably, while less critical tasks like user input don’t interfere.

## Watchdog Timer
In Progess
