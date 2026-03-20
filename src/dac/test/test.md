Author: Kenneth Dao
# Tests
## Verification Tests for InfluxDB3 and MQTT

This module provides end-to-end verification for two independent telemetry paths:

- **InfluxDB 3 write/read**: A strongly-typed telemetry packet is sent via `send_message`, then the InfluxDB3 SQL API is queried to confirm the most recent row matches the original packet.
- **MQTT publish/subscribe**: A listener subscribes to the `telemetry` topic, waits for the next publish, deserializes the payload into `TelemetryData`, and checks it matches the expected struct.

These tests validate both persistence (InfluxDB) and transport (MQTT) behavior using the same `TelemetryData` data model.

---

## Function: `verify_influx_write<T>(test_packet: T) -> Result<bool, Box<dyn Error>>`

Verifies that a packet written to InfluxDB can be read back and exactly matches the original struct.

### Behavior

- Creates an HTTP client using `reqwest`
- Builds a SQL query against the measurement returned by `T::topic()`
- Posts the query to the InfluxDB3 SQL endpoint  
  `POST {INFLUXDB_URL}/api/v3/query_sql`
- Deserializes the response into `Vec<T>`
- Compares the newest row against `test_packet`

### Requirements

- `T: Reading + Deserialize + PartialEq`
- A running InfluxDB3 instance
- Valid `INFLUXDB_URL` and `INFLUXDB_DATABASE`
- Authorization accepted by the InfluxDB3 instance

### SQL Query Used

```sql
SELECT * FROM <T::topic()> ORDER BY time DESC LIMIT 1