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
mosquitto_sub -h localhost -p 1883 -t tinkerforge/# -v
```

Switch on brennenstuhl via tinkerforge:
```bash
mosquitto_pub -t tinkerforge/request/remote_switch_bricklet/v17/switch_socket_a -m '{"house_code": 19, "receiver_code": 1, "switch_to": "on"}'
```

## Using the containerized setup

Thanks to [icebear8/tinkerforge](https://hub.docker.com/r/icebear8/tinkerforge) and [icebear8/mosquitto](https://hub.docker.com/r/icebear8/mosquitto) the above can be done easily using containers.

In this repo an example setup is provided (which fits a subset of my environment). The `docker-compose.yml` can be used and modified to fit your local environment. A simple command creates and starts the containers (which exposes the broker locally on port `1884`)
```bash
docker-compose up -d
```

Listen to all callbacks:
```bash
mosquitto_sub -h localhost -p 1884 -t tinkerforge/# -v
```

### Connecting with OpenHab
>   In my case, i wrote a `docker-compose.override.yml' to connect the services to the same (virtual) network in which my openhab instance resides (this depends on your concrete setup). 
> 
>    ```yaml
>    version: '3'
>    services:
>    mosquitto:
>        networks:
>        smarthome_net:
>            ipv4_address: 172.29.0.10
>    tinkerforge_j5005:
>        networks:
>        smarthome_net:
>            ipv4_address: 172.29.0.11
>    tinkerforge_slero:
>        networks:
>        smarthome_net:
>            ipv4_address: 172.29.0.12
>
>    networks:
>    smarthome_net:
>        external: true
>    ```


Inside the openhab container, the `mqtt.things` may define the broker and a thing defining the channels (each extracting the desired property with a `JSONPATH` transformation pattern):
```
Bridge mqtt:broker:broker [ host="172.29.0.10", secure=false ]
{
  Thing topic mytopics {
    Type string : temperature_j5005 "Temperatursensor Küche" [ stateTopic="tinkerforge/j5005/callback/temperature_bricklet/bP3/temperature", transformationPattern="JSONPATH:$.temperature" ]
    Type string : temperature_slero "Temperatursensor Slero" [ stateTopic="tinkerforge/slero/callback/temperature_bricklet/zrP/temperature", transformationPattern="JSONPATH:$.temperature" ]
  }
}
```

In `mqtt.items` items can be defined including a profile which e.g. divide the gotten value using a JS transformations: 
```
Number Temperature_j5005 "J5005 Temperature °C [%s °C]" <temp>  {channel="mqtt:topic:broker:mytopics:temperature_j5005"[profile="transform:JS", function="div100.js"]}
Number Temperature_slero "Slero Temperature °C [%s °C]" <temp>  {channel="mqtt:topic:broker:mytopics:temperature_slero"[profile="transform:JS", function="div100.js"]}
```

Such a JS file has to be placed in the `conf/transform/` folder, for our example named `div100.js`: 
```js
(function(i){
  return (parseFloat(i) / 100).toFixed(2).toString();
})(input)
```
