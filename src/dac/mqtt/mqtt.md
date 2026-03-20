Author: Lawrence Chan
# MQTT Broker

## Overview

`mqttd()` loads the embedded `rumqttd.toml` config, builds a `rumqttd::Broker`, and starts it.

---

## Function: `mqttd`

**Workflow:**

1. Reads `../rumqttd.toml` via `include_str!`.
2. Builds `config::Config` from TOML.
3. Deserializes into broker config.
4. Creates `Broker::new(config)` and calls `broker.start()`.

**Error Handling:**

- Logs and returns if config load fails.
- Logs and returns if config deserialization fails.
- Logs if broker startup fails.