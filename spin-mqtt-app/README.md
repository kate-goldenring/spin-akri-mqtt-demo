# MQTT Message Logger Spin App

Spin listens to an MQTT broker at a specific topic. It executes the Spin app for each newly published message. The Spin app simply logs the message.

## Running an MQTT broker

Download [MQTTX CLI](https://github.com/emqx/MQTTX/tree/main/cli)

```sh
brew install emqx/mqttx/mqttx-cli
```

Run the EMQX broker: https://mqttx.app/docs/get-started

```sh
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 emqx/emqx
```

Connect to locally running broker

```sh
mqttx conn -h '127.0.0.1' -p 1883
```

Subscribe to the hello topic

```sh
mqttx sub -t 'hello' -h '127.0.0.1' -p 1883
```

Publish to the hello topic

```sh
mqttx pub -t 'hello' -h '127.0.0.1' -p 1883 -m 'from me'
```


## Running the spin app to listen to a specific topic

```sh
SPIN_VARIABLE_MQTT_TOPIC="hello" SPIN_VARIABLE_MQTT_BROKER_URI="mqtt://localhost:1883" spin build --up 
```

Or use the pushed app:

```sh
SPIN_VARIABLE_MQTT_TOPIC="hello" SPIN_VARIABLE_MQTT_BROKER_URI="mqtt://localhost:1883" spin up --from ghcr.io/kate-goldenring/mqtt-message-logger:v1
```

Now publish a message to the hello topic:

```sh
mqttx pub -t 'hello' -h '127.0.0.1' -p 1883 -m 'from me'
```

The following should be logged in the `spin build --up` output:

```sh
"2024-06-07 20:21:32.265317000" Message received by wasm component: 'from me' on topic 'hello'
```
