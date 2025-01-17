# Frequently Asked Questions
* How do I **start** this *locally* on my pc?
    1. create and confgure an `.env` file. You can copy the default one as a starter `cp .env.default .env`.
    2. configure `src/data_collection/etc/cfg/dev/<blockchain>.json` (depending on your blockchain)
    3. run `bash scripts/run-dev-<blockchain>.sh`
* How do I **start** this on *Abacus-3*?
    1. create and confgure an `.env` file. You can copy the default one as a starter `cp .env.default .env`.
    2. configure `src/data_collection/etc/cfg/prod/<blockchain>.json` (depending on your blockchain)
    3. run `bash scripts/run-prod-<blockchain>.sh`
* How do I stop the process?
    * Use `KeyboardInterrupt` (`Ctrl+C`). Or `kill` if used in the bg.
    * After stopping, all the containers are stopped, but volume data is preserved.
* Is it possible to start multiple of these at the same time?
    * Yes, simply change the data directory (`DATA_DIR`) and the prefix of the containers (`PROJECT_NAME`) in the `.env` file.
* Does the run script **stop** / **cleanup** all the containers when the configured data collection is finished?
    * Yes. The consumers wait 5 minutes after the last received event before shutting themselves down. Producer closes immediately after the collection process has been finished. Other containers close when consumers and producers are down.
* **How many topics and consumers** should I use?
    * Depends on the machine you're running on, but generally the more consumers and topics, the faster the processing.
* Why does the production environment add an **Erigon proxy service** instead of just using `host.docker.internal` within the consumers / producers?
    * This reverse proxy is used as a workaround for the abacus-3 firewall. Currently the firewall rules only allow the default docker0 interface to send outbound requests to the host machine. This means that any container started with `docker run --add-host=host.docker.internal:host-gateway ...` will be able to reach the host. This works because the default docker network used for any `docker run` is the 'bridge' which has its default gateway set to the docker0 interface address. However, as soon as you attempt to do this inside of a docker compose (`extra_hosts: host.docker.internal:host-gateway`), it stops working. This is because the compose creates a separate docker network which has a random gateway address used for communication with the host. In theory, you could [set up a custom docker network with a static gateway address](https://stackoverflow.com/a/60245651/4249857) inside of the compose and add this address to the firewall rules, but that wasn't possible at the time of development.
* Why am I seeing `Unable connect to "kafka:9092": [Errno 111] Connect call failed ('<container_ip>', 9092)` in the logs?
    * The producers and consumers attempt to connect to the Kafka container as soon as they're started. However the Kafka container takes some time and is usually <15s but sometimes it takes a bit longer. These messages are generated by the internal kafka library that is used within the project and can be ignored.
* Why am I seeing `Group Coordinator Request failed: [Error 15] CoordinatorNotAvailableError` in the logs?
    * Another Kafka internal log that can be ignored, the coordinator is eventually selected and this error is irrelevant.
* Why am I seeing `Heartbeat failed for group eth because it is rebalancing` in the logs?
    * Another Kafka internal log that can be ignored. Happens when the number of consumers changes because kafka has to rebalance the partitions for a topic.
