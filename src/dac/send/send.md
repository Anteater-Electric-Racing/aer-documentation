Author: Alistair Keiller
# Telemetry Fan-Out (MQTT + TDengine)

## Overview

`send_message()` serializes a telemetry reading once, attaches a `ts` field, then fans out to both MQTT and TDengine.

---

## Function: `send_message`

**Workflow:**

1. Serialize message to JSON; insert `ts` with the provided timestamp.
2. Publish JSON to MQTT topic `T::topic()` at QoS `AtMostOnce`.
3. Convert to TDengine schemaless line protocol and enqueue for write.

**Backpressure / error behavior:**

- Serialization failure → log, drop
- MQTT queue full → warn, drop; publish error → log
- Line protocol conversion failure → log, drop
- TDengine queue full → warn, drop; channel error → log

---

## Runtime Singletons

**`get_mqtt_client`** — lazily creates an `AsyncClient` (`OnceCell`), spawns the MQTT event loop. On poll error: logs and retries after 1 second.

**`get_tdengine_sender`** — lazily creates a channel sender (`OnceCell`), ensures the `fsae` database exists, and spawns 4 worker tasks sharing one receiver. Workers batch up to 5,000 records per write. After 10 consecutive write failures, drops and recreates the database.

---

## Trait: `Reading`

Requires `Serialize`. Implementors define `topic() -> &'static str` to specify the MQTT/TDengine measurement name.

---

## Function: `now_ms`

Returns current UNIX time as milliseconds (`u64`).