# Argon ONE Lite

## Installed files

- `/usr/local/sbin/argonone-lite`
- `/etc/argonone-lite.conf`
- `/etc/systemd/system/argonone-lite.service`

## Status

```bash
sudo systemctl status argonone-lite.service
sudo journalctl -u argonone-lite.service -n 50 --no-pager
```

## Edit fan thresholds

Edit `/etc/argonone-lite.conf`, then restart the service:

```bash
sudo systemctl restart argonone-lite.service
```

Threshold entries are under `[fan]` and use this format:

```ini
temp_60 = 10
temp_65 = 50
temp_72 = 100
```

This means:

- below 60C = 0%
- 60C and above = 10%
- 65C and above = 50%
- 72C and above = 100%

## MQTT status

MQTT publishing is optional and is configured in `/etc/argonone-lite.conf` under `[mqtt]`:

```ini
[mqtt]
mqtt_enabled = true
mqtt_host = localhost
mqtt_port = 1883
mqtt_topic = moho/argonone/status
mqtt_client_id = argonone-lite
mqtt_retain = true
mqtt_publish_interval_seconds = 30
mqtt_discovery_enabled = true
mqtt_discovery_prefix = homeassistant
mqtt_device_id = moho_argonone
mqtt_device_name = MoHo Argon ONE
mqtt_manufacturer = Argon Forty
mqtt_model = Argon ONE V1
```

The daemon publishes retained JSON status to `moho/argonone/status` on the host-local broker at `localhost:1883`.
Home Assistant discovery config is published to `homeassistant/device/moho_argonone/config`.

Example payload:

```json
{"temperature_c":46.74,"fan_percent":0,"fan_state":"off","fan_target_temperature_c":60.0,"uptime_seconds":42,"service_version":"1.2.0","hostname":"moho-docker-rpi","timestamp":"2026-07-09T20:52:49.744000Z","button_monitor":"shutdown-only","mqtt_discovery_enabled":true,"mqtt_discovery_prefix":"homeassistant"}
```

Discovered Home Assistant entities:

- `sensor.moho_argon_one_cpu_temperature`
- `sensor.moho_argon_one_fan_speed`
- `sensor.moho_argon_one_next_fan_threshold`
- `sensor.moho_argon_one_daemon_uptime`
- `sensor.moho_argon_one_service_version`
- `sensor.moho_argon_one_fan_state`
- `sensor.moho_argon_one_button_monitor`

Manual Home Assistant YAML is no longer required when discovery is enabled, but this example shows the source data shape:

```yaml
mqtt:
  sensor:
    - name: "MoHo Argon CPU Temperature"
      unique_id: moho_argon_cpu_temperature
      state_topic: "moho/argonone/status"
      unit_of_measurement: "C"
      device_class: temperature
      value_template: "{{ value_json.temperature_c }}"

    - name: "MoHo Argon Fan Percent"
      unique_id: moho_argon_fan_percent
      state_topic: "moho/argonone/status"
      unit_of_measurement: "%"
      value_template: "{{ value_json.fan_percent }}"

    - name: "MoHo Argon Fan State"
      unique_id: moho_argon_fan_state
      state_topic: "moho/argonone/status"
      value_template: "{{ value_json.fan_state }}"
```

## Discovery topics

- Status topic: `moho/argonone/status`
- Discovery topic: `homeassistant/device/moho_argonone/config`
- Home Assistant birth topic watched by the daemon: `homeassistant/status`

The daemon publishes retained discovery config on startup, on MQTT reconnect, and again when Home Assistant publishes `online` to `homeassistant/status`.

## Disable discovery

To stop publishing discovery config but keep normal status MQTT:

```ini
[mqtt]
mqtt_discovery_enabled = false
```

Then restart:

```bash
sudo systemctl restart argonone-lite.service
```

This stops future discovery updates, but existing discovered entities may remain in Home Assistant until removed there or until the retained discovery topic is cleared.

If you need to clear the retained discovery payload from the local Mosquitto container:

```bash
docker exec mosquitto-MQTT mosquitto_pub -h 127.0.0.1 -p 1883 -t homeassistant/device/moho_argonone/config -r -n
```

## Troubleshooting

- Check the daemon logs with `sudo journalctl -u argonone-lite.service -n 100 --no-pager`.
- Check that the local broker is listening on `localhost:1883`.
- Verify the retained status message:
  `docker exec mosquitto-MQTT mosquitto_sub -h 127.0.0.1 -p 1883 -t moho/argonone/status -C 1`
- Verify the retained discovery message:
  `docker exec mosquitto-MQTT mosquitto_sub -h 127.0.0.1 -p 1883 -t homeassistant/device/moho_argonone/config -C 1`
- If Home Assistant misses discovery after a restart, confirm that its MQTT integration discovery prefix is still `homeassistant`.

## Restart

```bash
sudo systemctl restart argonone-lite.service
```

## Uninstall

```bash
sudo systemctl disable --now argonone-lite.service
sudo rm -f /etc/systemd/system/argonone-lite.service
sudo rm -f /usr/local/sbin/argonone-lite
sudo rm -f /etc/argonone-lite.conf
sudo systemctl daemon-reload
```

## Known limitations

- Fan control uses the Argon ONE controller at I2C address `0x1a` with small direct writes only when the target speed changes.
- Button handling watches GPIO4 for the pulse pattern used by the Argon ONE case.
- This build intentionally enables shutdown handling only by default. Reboot pulse handling is left disabled in config unless explicitly wanted.
- MQTT publishing is best-effort only. Fan and button control continue even if the broker is unavailable or publishes fail.
- Physical button behavior was not triggered remotely during deployment, so the button path is based on inspected public implementation details plus local GPIO/I2C evidence rather than a live press test.
