# INSTALL

## Prerequisites

- Raspberry Pi with an Argon ONE V1-compatible controller visible on I2C bus 1
- Python 3
- `python3-smbus2`
- `python3-libgpiod`
- `python3-paho-mqtt` if MQTT publishing is wanted
- systemd

On the tested host, the Argon controller was visible at I2C address `0x1a`.

## Enable I2C

Make sure I2C is enabled for the Raspberry Pi and that `/dev/i2c-1` exists.

Typical checks:

```bash
ls -l /dev/i2c-1
i2cdetect -y 1
```

Expected Argon ONE controller address:

```text
0x1a
```

## Copy files into place

From this release folder on the Pi:

```bash
sudo install -m 0755 argonone-lite /usr/local/sbin/argonone-lite
sudo install -m 0644 argonone-lite.conf.example /etc/argonone-lite.conf
sudo install -m 0644 argonone-lite.service /etc/systemd/system/argonone-lite.service
```

If `/etc/argonone-lite.conf` already exists and you want to keep local settings, merge changes instead of overwriting it.

## Edit config

Review:

```bash
sudoedit /etc/argonone-lite.conf
```

Common settings:

- Fan thresholds under `[fan]`
- MQTT enable/disable and broker details under `[mqtt]`
- Home Assistant discovery enable/disable under `[mqtt]`
- `shutdown_only = true` under `[button]`

## Enable and start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now argonone-lite.service
```

## Check logs and status

```bash
sudo systemctl status argonone-lite.service
sudo journalctl -u argonone-lite.service -n 100 --no-pager
```

## Home Assistant discovery notes

If `mqtt_discovery_enabled = true`, the daemon publishes retained discovery to:

```text
homeassistant/device/moho_argonone/config
```

Default Home Assistant MQTT integration discovery prefix should remain:

```text
homeassistant
```

The daemon also listens for Home Assistant MQTT birth on:

```text
homeassistant/status
```

On startup and reconnect, it republishes discovery and the latest retained status.

## Safety note

This is a community/independent lightweight daemon and is not affiliated with Argon Forty.

Only test power-button behavior when it is safe for the system to shut down.
