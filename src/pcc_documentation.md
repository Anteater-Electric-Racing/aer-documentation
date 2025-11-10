# Subsystem + Purpose (Anna)

# Pre-Charge Circuit

---

The pre-charge circuit is a subsystem that ensures that the accumulator (battery) will charge safely by limiting the inrush current when the car is first powered on. This protects the rest of the system from potential damage.

# How it works, i/o (Adam)

# Important Modules

### General Overview (Anna)

As mentioned in HV, the PCC is constantly reading the accumulator voltage and tractive system voltage and printing them to the serial monitor. As it does so, it transitions between four possible states: **STANDBY**, **PRECHARGE**, **ONLINE**, and **ERROR**.

1. STANDBY (**void standby()**): The initial/idle state of the PCC. Waits for a stable shutdown circuit (SDC). Opens accumulator isolation relays (AIR) and precharge relay. If the accumulator voltage is greater than or equal to the minimum voltage for the shutdown circuit (**PCC_MIN_ACC_VOLTAGE**), transitions into the PRECHARGE state.
2. PRECHARGE (**void precharge()**): Closes AIR- and precharge relay. Monitors precharge progress, which is a function of the tractive system voltage and accumulator voltage. If the precharge progress is completed (prechargeProgress >= **PCC_TARGET_PERCENT**), checks if the target percentage was reached too quickly. If so, transitions into the ERROR state. Otherwise, transitions into the ONLINE state. If the precharge is too slow, it will also transition into the ERROR state.
3. ONLINE (**void running()**): The status that indicates that precharge has safely and successfully completed. Closes AIR+ to connect accumulator (ACC) to tractive system (TS) and opens precharge relay.
4. ERROR (**void errorState()**): The PCC error status. Opens AIRs and precharge relay.

### main.cpp

##### void setup()

Initializes GPIO pins and the precharge system.

##### void threadMain()

Continuously reads accumulator voltage and tractive system voltage and switches between the four states based on corresponding conditions.

### precharge.cpp

Defines states creates low pass filter.

##### getFrequency(int pin)

Gets frequency of a square wave signal from a GPIO pin.

##### getVoltage(int pin)

Converts the frequency into voltage for accumulator or tractive system GPIO pins.

##### prechargeINIT()

Initializes the mutex and precharge task.

##### prechargeTask(void \*pvParameters)

Handles the state machine and status updates.

##### void standby(), void precharge(), void running(), void errorState(). See General Overview.

##### float getTSVoltage()

Gets the tractive system voltage.

##### float getAccumulatorVoltage()

Gets the accumulator voltage.

##### PrechargeState getPrechargeState()

Returns the current precharge state (undefined, standby, precharge, online, or error).

##### getPrechargeError()

Returns current error information as an error code.

### gpio.cpp

Initializes GPIO pins (shutdown control pin, accumulator pin, tractive system pin).

### can.cpp

CAN communication.

# Improvements + next steps

- Clean up code (get rid of duplicate and unncessary comments)
