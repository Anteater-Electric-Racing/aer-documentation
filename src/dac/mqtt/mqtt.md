# MQTT

Author: Lawrence Chan

## CAN-to-InfluxDB Telemetry Pipeline via MQTT

### Overview

This system reads data from a CAN bus, sends it through MQTT, verifies the packet, and stores it in InfluxDB.

---

### Workflow

1. **CAN Bus Data Collection**
   - Reads telemetry values such as motor speed, torque, voltage, current, temperatures, and fault states.
   - Packages data into a `TelemetryData` structure.

2. **MQTT Transmission**
   - Publishes `TelemetryData` to the `"telemetry"` topic on the MQTT broker.
   - Ensures reliable delivery using QoS `AtLeastOnce`.

3. **MQTT Listener Verification**
   - Subscribes to the `"telemetry"` topic.
   - Converts incoming MQTT payload bytes to UTF-8 strings.
   - Deserializes JSON into `TelemetryData`.
   - Compares received data against expected telemetry for verification.
   - Returns `true` if verification succeeds.

4. **InfluxDB Storage**
   - On successful verification, telemetry is stored in InfluxDB.
   - Each parameter (e.g., motor speed, voltage, fault flags) is stored as a separate measurement or field for time-series tracking.

---

### Key Features

- **Real-Time Monitoring**: Data flows from CAN bus → MQTT → verification → InfluxDB.
- **Error Handling**: Logs failed subscriptions, parsing errors, or deserialization issues.
- **Test Coverage**: Includes asynchronous tests to ensure listener correctly receives and verifies messages.
- **Extensible**: Can add new telemetry fields or modify verification rules without disrupting the pipeline.

------

## MQTT Listener Verification Summary

### Function: `verify_mqtt_listener`

- **Purpose**: Verifies that an MQTT listener receives a specific telemetry message.
- **Parameters**: `telemetry_struct: TelemetryData` – the expected telemetry data to verify against.
- **Returns**: `bool` – `true` if the message is received and matches, otherwise `false`.

#### Workflow

1. **Setup MQTT Client**
   - Creates an `AsyncClient` with a 5-second keep-alive.
   - Connects to `MQTT_HOST` and `MQTT_PORT`.

2. **Subscribe to Topic**
   - Subscribes to `"telemetry"` topic with QoS `AtLeastOnce`.
   - Logs an error and returns `false` if subscription fails.

3. **Listen for Messages**
   - Polls the event loop for incoming packets.
   - Timeout: 1 second.
   - On `Publish` packets:
     - Converts payload bytes to a UTF-8 string.
     - Deserializes JSON into `TelemetryData`.
     - Returns `true` if it matches `telemetry_struct`.

4. **Error Handling**
   - Logs conversion or deserialization errors.
   - Returns `false` if no matching message is received or timeout occurs.

---

### Test: `test_verify_mqtt_listener`

- **Purpose**: Ensures `verify_mqtt_listener` correctly detects the telemetry message.

#### Steps

1. **Prepare Test Data**
   - Creates `TelemetryData` with default values and states.

2. **Run Listener**
   - Initializes a Tokio runtime.
   - Spawns `verify_mqtt_listener` asynchronously.

3. **Send Test Message**
   - Sleeps for 500 ms to allow the listener to start.
   - Sends the test telemetry message via `send_message`.

4. **Validate**
   - Waits for listener result.
   - Asserts that the listener received the expected message.
   - Logs failure if the message was not received.

---

**Key Points**

- Uses asynchronous MQTT client (`rumqttc::AsyncClient`).
- Handles message parsing, deserialization, and matching safely.
- Includes robust error logging for debugging.
- Test ensures integration works in a runtime environment.
