# UNINSTALL

Stop and disable the service:

```bash
sudo systemctl disable --now argonone-lite.service
```

Remove installed files:

```bash
sudo rm -f /etc/systemd/system/argonone-lite.service
sudo rm -f /usr/local/sbin/argonone-lite
sudo rm -f /etc/argonone-lite.conf
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Optional retained MQTT discovery cleanup:

```bash
docker exec mosquitto-MQTT mosquitto_pub -h 127.0.0.1 -p 1883 -t homeassistant/device/moho_argonone/config -r -n
```

Optional retained MQTT status cleanup:

```bash
docker exec mosquitto-MQTT mosquitto_pub -h 127.0.0.1 -p 1883 -t moho/argonone/status -r -n
```

If you remove retained discovery messages, Home Assistant may also need entity cleanup from its UI depending on prior discovery state.
