Author: Kenneth Dao
# Send
## MQTT + InfluxDB Telemetry Dispatch

Publishes strongly-typed telemetry readings to MQTT and persists the same data to InfluxDB3.  
The key entry point is `send_message`, which ensures every `Reading` is delivered to real-time
subscribers and written to the database using a consistent topic/measurement name.

---

## Trait: `Reading`

Defines the contract for any telemetry type that can be dispatched through the system.

- Serialization: Requires `Serialize` for JSON encoding and line protocol conversion.
- Topic: `topic()` returns the MQTT topic and InfluxDB measurement name.

This guarantees topic consistency across transport and persistence layers.

---

## Function: `send_message<T: Reading>(message: T)`

- MQTT: Publishes telemetry as JSON to `T::topic()` with QoS `AtLeastOnce`.
- InfluxDB: Converts telemetry to line protocol and writes it to the database.
- Ordering: MQTT publish is attempted before the InfluxDB write.
- Fault tolerance: Failures in one path do not prevent attempts in the other.

### MQTT Publish

- Client: Shared `rumqttc::AsyncClient`
- Topic: `T::topic()`
- QoS: `AtLeastOnce`
- Retain: `false`
- Payload: JSON serialization of `message`

### InfluxDB Write

- Conversion: Uses `to_line_protocol(&message)`
- Endpoint:  
  `POST /api/v3/write_lp?db=<db>&precision=nanosecond`
- Body: Line protocol string
- Auth: `Authorization: Bearer <token>`

---

### Workflow Inside `send_message`

1. Serialize `message` to JSON (`serde_json::to_string`).
2. Publish JSON payload to MQTT on `T::topic()`.
3. Convert `message` to InfluxDB line protocol.
4. POST line protocol to the InfluxDB write endpoint.
5. Log errors without panicking to maintain system operation.

---

## Client Initialization

### MQTT Client

- Lazily initialized using `OnceCell`
- Uses a background event loop task to:
  - Maintain keep-alive
  - Handle acknowledgments
  - Recover from transient broker errors

### InfluxDB Client

- Lazily initialized `reqwest::Client`
- Shared across all write operations
- Avoids repeated client construction under load

---

## End-to-End Flow

1. Collect CAN telemetry into a struct implementing `Reading`.
2. Call `send_message(telemetry)`.
3. Telemetry is:
   - Published as JSON to MQTT subscribers.
   - Persisted to InfluxDB as line protocol.
4. Downstream systems can:
   - Consume real-time data via MQTT.
   - Query historical data from InfluxDB.

---

## Writing to InfluxDB

- Endpoint: `POST /api/v3/write_lp`
- Query Parameters:
  - `db=<database>`
  - `precision=nanosecond`
- Body: Output of `to_line_protocol`
- Auth: Bearer token header
- Reliability:
  - Write failures are logged
  - System continues operating on partial failure

Minimal sketch:

1. `send_message(telemetry).await;`
2. MQTT publish and InfluxDB write occur automatically.

---

## Testing Guidance

- Unit tests:
  - Verify `Reading::topic()` consistency.
  - Ensure JSON serialization succeeds.
- Integration tests:
  - Pair with MQTT listener verification.
  - Confirm InfluxDB writes can be queried back.

---

## Troubleshooting

- MQTT publish failures:
  - Check broker availability and credentials.
  - Verify `MQTT_HOST` and `MQTT_PORT`.
- InfluxDB write failures:
  - Confirm database name and write endpoint.
  - Validate bearer token and server availability.
- Missing telemetry:
  - Ensure the telemetry struct implements `Reading`.
  - Verify `topic()` matches expected measurement/topic.