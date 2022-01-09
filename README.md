# Tinkerforge with MQTT 

## Setup and Testing

Search for available brick daemons (`brickd`) in the network
```bash
nmap -p4218-4223 192.168.0.0/24
```

Use the brick viewer `brickv` to check for available bricks/bricklets to configure the `<init>,.json` file(s).

Translation proxy to publish events on an mqtt broker (e.g. using mosquitto)
```bash
tinkerforge_mqtt --ipcon-port 4220 --broker-port 1883 --init-file init.json 
```

Listen to all callbacks:
```bash
mosquitto_sub -h localhost -p 1883 -t tinkerforge/callback/#
```

Switch on brennenstuhl via tinkerforge:
```bash
mosquitto_pub -t tinkerforge/request/remote_switch_bricklet/v17/switch_socket_a -m '{"house_code": 19, "receiver_code": 1, "switch_to": "on"}'
```

## Using the containerized setup

Thanks to [icebear8/tinkerforge](https://hub.docker.com/r/icebear8/tinkerforge) and [icebear8/mosquitto](https://hub.docker.com/r/icebear8/mosquitto) the above can be done easily in containers.
In this repo an example setup is provided (which fits a subset of my environment). The `docker-compose.yml` can be used and modified to fit your local environment. 


