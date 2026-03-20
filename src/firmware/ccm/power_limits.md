# PowerLimit_Update()
## Purpose 
This function ensures the car does not request motor torque which exceeds two important torque threshold requirements. The two cases in which this function would intervene are when max requested power exceeds 80kW (EV.3.3.1) and when the accumulator SOC is < 20%, in which rules specify that we must linearly reduce allowed power down to 0 at 5%. 

## Structure 

Given above is a block diagram of the PowerLimit_update() function. In our codebase, PowerLimit_update() is called inside motor.cpp, during Motor_UpdateMotor()’s MOTOR_STATE_DRIVING state. The variable torqueDemand is given as an argument to Motor_UpdateMotor(), and this is then passed into our function, and finally returned and assigned to motorData.desiredTorque. A function somewhere else in the codebase will then read this value and send it to the motors, creating the actual torque requested.  

In addition to torqueDemand, PowerLimit_Update() requires accumulator voltage and current as well as accumulator SOC. Voltage and current are multiplied to find power.  

## Implementation 
There are a few constants used in this method, all of which are defined inside /utils/utils.h. What these all represent and what they are used for are as follows: 

SOC_START, SOC_END 

Represents the percentage (as a decimal) at which SOC-based torque limiting (derating) begins and ends. As of 10/2025 they are set to .2 and .05, respectively. Derating will begin at SOC_START, and will linearly reduce until reaching SOC_END, when torqueDemand will be 0.  

MOTOR_MAX_TORQUE 

Represents the maximum torque that the car is designed to output. Measured in N*m. This is used to linearly scale the torqueLimit proportionally based on SOC.  

POWER_THRESHOLD_kWh 

Represents maximum power allowed in FSAE rulebook. When calculated power is over this amount, driverTorqueCmd gets scaled down by this threshold divided by the calculated power. The assumption behind this is that power and torque are proportional, meaning that when calculated power is 10% above the threshold, driverTorqueCmd gets scaled down accordingly by 10%.  

SOC derating amount is calculated and then stored inside the local variable torqueLimit, which is assigned to driverTorqueCmd at the end. Power limiting doesn’t use torqueLimit, it just directly sets driverTorqueCmd.  

## Testing 
Method signature:  

PowerLimit_Update(float soc, float voltage, float current, float driverTorqueCmd) 

Constants: 

MOTOR_MAX_TORQUE = 260.0F 

POWER_THRESHOLD_kWh = 90  

SOC_START = 0.2  

SOC_END = 0.05  

Test case 1 (base case): 

PowerLimit_Update(.25, 40, 2, 200); 

Returns 200 

Test case 2 (MOTOR_MAX_TORQUE condition checking): 

PowerLimit_Update(.25, 40, 2, 270); 

Returns 260 

Test case 3 (POWER_THRESHOLD_kWh condition checking): 

PowerLimit_Update(.25, 40, 3, 200); 

Returns 150 

Test case 4 (SOC derating checking): 

PowerLimit_Update(.1, 40, 2, 200); 

Returns 66.67 

Test case 5 (SOC end derating checking): 

PowerLimit_Update(.05, 40, 2, 200); 

Returns 0 