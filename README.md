# Akri and Spin Demo

Akri with MQTT Discovery Handler and Spin App Broker.

1. Install kwasm
2. Replace the installed `containerd-shim-spin-v2` with one with support for the Spin MQTT trigger: https://github.com/kate-goldenring/containerd-shim-spin/tree/mqtt-trigger:

    ```sh
    git clone git@github.com:kate-goldenring/containerd-shim-spin.git
    cd containerd-shim-spin
    git checkout mqtt-trigger
    cd containerd-shim-spin && cargo build --release
    # Copy the shim to where the containerd shims are. For example:
    sudo cp target/release/containerd-shim-spin-v2 /usr/bin/
    ```

    > Note: this branch of the shim references an [updated MQTT trigger](https://github.com/kate-goldenring/spin-trigger-mqtt/tree/use-spin-telemetry) with the latest Spin dependencies

3. Install Akri with the updated MQTT Discovery Handler and updated Akri Agent
   Build containers for the Agent and MQTT DH from the following branches
      - Updated Akri Agent without Broker Properties Suffix (no instance ID added to broker ENV Vars) and updated discovery proto without optional field (to be compatible with MQTT DH: https://github.com/kate-goldenring/akri/tree/mqtt-demo-changes
      - Updated MQTT DH (to add SPIN_VAR prefix to the broker properties): https://github.com/kate-goldenring/akri-mqtt-discovery-handler/tree/mqtt-on-spin

    Install Akri referencing the built images:
    ```sh
    helm repo add akri-helm-charts https://project-akri.github.io/akri/
    helm upgrade akri akri-helm-charts/akri \
    $AKRI_HELM_CRICTL_CONFIGURATION \
    --set agent.image.repository=ghcr.io/kate-goldenring/agent \
    --set agent.image.tag=spin-demo \
    --set custom.discovery.enabled=true  \
    --set custom.discovery.image.repository=ghcr.io/myusername/mqtt-discovery-handler \
    --set custom.discovery.image.tag=v2 \
    --set custom.discovery.name=akri-mqtt-discovery 
    ```

4. Apply the Akri Configuration to kick off discovery

    ```sh
    kubectl apply -f deploy/akri-mqtt-spin-configuration.yaml
    ```

5. Run an MQTT broker in the cluster. For example, to run locally:

    ```sh
    docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 emqx/emqx
    ```

6. Regularly publish messages to the broker

    ```sh
    for i in $(seq 1 100000) ; do mqttx pub -t 'hello' -h '127.0.0.1' -p 1883 -m "from MQTTX CLI $i" && sleep 10 ; done
    ```

7. An Akri instance should be discovered for the "hello" topic and a Spin app broker created!