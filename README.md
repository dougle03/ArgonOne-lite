# argonone-lite

`argonone-lite` is a small, host-focused Python daemon for Argon ONE V1 fan control and front power-button monitoring on Raspberry Pi.

It was built as a lightweight alternative to the broader official installer flow. The aim is to keep the install readable, reversible, and easy to audit: one daemon, one config file, one systemd unit, and journal logging.

<img width="1542" height="1134" alt="image" src="https://github.com/user-attachments/assets/95029831-47a2-45ce-8c3e-d1b48ad47171" />

Independent implementation rather than wrapping the vendor installer.
Minimal attack surface—one daemon, one config, one service.
No dependency on Argon's download server after installation.
Home Assistant native integration via MQTT Discovery.
Systemd-native with journald logging.
Documented and versioned from day one.

Tested hardware and OS:

- Raspberry Pi 4 8GB
- Argon ONE V1 case
- Debian GNU/Linux 13 (Trixie), 64-bit / aarch64

It may also work on similar Argon ONE Raspberry Pi 4 cases, but this project is only tested on the hardware and OS listed above.

## Features

- Temperature-based fan control over the Argon ONE I2C controller at `0x1a`
- Front power-button monitoring with shutdown-focused handling
- Optional MQTT status publishing
- Optional Home Assistant MQTT Discovery
- Small, readable, systemd-friendly deployment

## Why not the official installer?

This project exists for users who want a narrower install surface than the vendor script. It avoids a broad setup flow that can bundle unrelated downloads, wider filesystem changes, and package upgrade behavior that may be undesirable on an existing Docker host or appliance-style Pi.

## Files

- `argonone-lite`
- `argonone-lite.conf.example`
- `argonone-lite.service`
- `INSTALL.md`
- `UNINSTALL.md`
- `CHANGELOG.md`
- `LICENSE`

## MQTT status

Default status topic:

- `moho/argonone/status`

Default discovery topic:

- `homeassistant/device/moho_argonone/config`

Status payload fields include:

- `temperature_c`
- `fan_percent`
- `fan_state`
- `fan_target_temperature_c`
- `uptime_seconds`
- `service_version`
- `hostname`
- `timestamp`
- `button_monitor`
- `mqtt_discovery_enabled`
- `mqtt_discovery_prefix`

## Safety note

This is a community-maintained independent lightweight daemon. It is not affiliated with or endorsed by Argon Forty.

Power-button behavior should only be tested when it is safe for the Raspberry Pi to shut down cleanly.
