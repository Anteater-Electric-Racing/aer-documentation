Author: Sebastian Ethan Basa
# ORION BMS (Battery Management System)

1[" "](../../images/firmware/OrionBMS.jpg)

The purpose of the BMS is to monitor various aspects of the battery, protect its lifespan, and report any faults with the battery.

![" "](../../images/firmware/BMS-Code.png)

The BMS transmits data via CAN messages which have their own unique CAN IDs. Using C++ and FreeRTOS, we can retrieve the CAN data from the BMS in the form of data structs, which can be interpreted and used throughout the code to control the car. 
Some of the data values that the BMS can transmit include battery cell voltages, temperatures, battery health, and fault conditions.
