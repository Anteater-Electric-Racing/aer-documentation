Author: Alistair Keiller
# InfluxDB
## CAN-to-InfluxDB Telemetry (Line Protocol)

Transforms strongly-typed telemetry readings into InfluxDB Line Protocol after MQTT verification. The key helper is `to_line_protocol`, which turns any `Reading` into a write-ready line.

---

## Function: `to_line_protocol<T: Reading>(message: &T) -> String`

- Measurement: Uses `T::topic()` as the measurement name.
- Fields: Serializes number, string, and boolean fields; unsupported types are skipped.
- Timestamp: Appends current UTC time in nanoseconds.

### Field Formatting

- Integers (`i64`, `u64`): Suffixed with `i`, e.g., `rpm=1200i`.
- Floats: Written as-is, e.g., `voltage=12.34`.
- Strings: Quoted, e.g., `state="OK"`.
- Booleans: Lowercase `true`/`false`.

### Example Output

- Input: `TelemetryData { rpm: 1200, voltage: 12.34, state: "OK", fault: false }`
- Topic: `telemetry`
- Output: `telemetry rpm=1200i,voltage=12.34,state="OK",fault=false 1734043200000000000`

### Workflow Inside `to_line_protocol`

1. Start with `T::topic()` as the measurement.
2. Serialize `message` to JSON (`serde_json::to_value`).
3. Iterate object fields, formatting supported values and comma-separating them.
4. Append the UTC nanosecond timestamp from `chrono::Utc::now().timestamp_nanos_opt()`.
5. Return the completed line protocol string.

---

## End-to-End Flow

1. Collect CAN telemetry (speed, torque, voltage, current, temps, fault states) into a `TelemetryData` that implements `Reading`.
2. Publish the struct as JSON to the `telemetry` MQTT topic with QoS `AtLeastOnce`.
3. MQTT listener verifies the payload matches the expected `TelemetryData`.
4. On verification success, convert to line protocol via `to_line_protocol` and write to InfluxDB.

---

## Writing to InfluxDB

- Endpoint: HTTP `POST /api/v2/write?org=<org>&bucket=<bucket>&precision=ns` or the official Influx client.
- Body: The string returned by `to_line_protocol`.
- Auth: `Authorization: Token <token>` header.
- Precision: `ns` to match the timestamp source.
- Reliability: Add retries/backoff for transient failures and log write errors.

Minimal sketch:

1. `let lp = to_line_protocol(&telemetry);`
2. POST `lp` to the write endpoint with `precision=ns`.

---

## Testing Guidance

- Unit tests: verify integer `i` suffixing, string quoting, boolean casing, and timestamp inclusion.
- Integration: pair with the MQTT listener test to cover publish → verify → convert → write.

---

## Troubleshooting

- Missing fields: Ensure fields are serde-serializable scalars (no nested objects/arrays).
- Wrong measurement: Confirm `T::topic()` returns the intended measurement.
- Write failures: Check token, bucket, org, precision, and server availability.
- Timestamp `None`: Verify system clock and `timestamp_nanos_opt()` support on the target platform.