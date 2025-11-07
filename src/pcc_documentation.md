# Subsystem + Purpose (Anna)
# Pre-Charge Circuit

---

The pre-charge circuit is a subsystem that ensures 


# Hardware
## Inputs 
TS+/-: positive and negative terminals of the tractive system
B+_in: positive terminal of accumulator
Shutdown_in: active high shutdown signal that triggers safe discharging of tractive system

## Outputs
Shutdown_in: propagates the shutdown signal to other circuits
IR+_GND: (unsure)
CANH/CANL: for communication purposes with other modules

## How It Works
The PCC is split into two halves, high voltage (HV) and low voltage (LV). The HV side deals with accumulator and tractive system voltages which routinely go up to 400 V, while the LV side does not exceed 12 V. The FSAE rulebook specifies that there must be separation between these two voltage levels. However, we still need to communicate betweeen the two sides, as the teensy needs to read the voltages of accumulator and TS, both of which are on the HV side. In our circuit, this is accomplished using two optocouplers. Optocouplers achieve voltage separation utilizing an LED on one side (in this case HV) which will light up when there is a signal, and a phototransistor on the other side which will react to that LED.

### HV Side
The PCC is constantly reading accumulator voltage and tractive system voltage. These are the main inputs to the system, and are fed into the board with a 2x2 molex connector (J6). Although the positive accumulator input is labeled as B+_in, on the schematic the signal that is fed into the V2F converter is called Vin_Accu. This is just a labeling difference, however, because when the precharge relay is closed we can clearly see that Vin_Accu would be the same voltage as B+_in. To read the voltages, the circuit utilizes two voltage to frequency converters, with input voltages that are stepped down with a voltage divider resistor network to be 1/10 of the actual value. 

There are two main relays in the PCC: Precharge Relay and Discharge Relay, both of which are controlled by the Shutdown_in signal. When both relays are closed, the tractive system is connected to the accumulator in parallel with two series 500 Ohm resistors, allowing for safe charging. In the case of a shutdown, both relays will be opened, (insert explanation of discharge, not sure currently how it works safely). 

Most of the resistors and complexity of the circuit are for the V2F converters, and those largely don't need to be worried about. What is important to understand here is that the V2F converters convert an analog voltage to a signal with a frequency proportional to that voltage. Meaning: the V2F output signal is oscillating between 1 and 0. When vin is high, that oscillation will be faster, and when it is low that oscillation will slow down. Then, using the optocouplers to maintain separation, that signal is fed to the teensy which then samples that data, and is able to control the rest of the circuit. The detailed implementation of these LM331 V2F converters involve capacitors and resistors which should be tuned to special values, but for this discussion of the PCC we will not dive into that. 


# Important modules (Anna)

# Improvements + next steps

## Anna does code, Adam does circuit
