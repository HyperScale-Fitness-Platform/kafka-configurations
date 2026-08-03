# Kafka Setup — HyperScale Fitness

This runs a single-broker Kafka cluster (using KRaft mode, no Zookeeper needed) plus a web UI to inspect topics and messages.

## What's included

- **kafka** — the Kafka broker itself, listening on port `9092` for your services
- **kafka-ui** — a web dashboard to view topics, messages, and consumer groups, available at `http://localhost:7070`

## Prerequisites

- Docker and Docker Compose installed
- Ports `9092`, `9093`, and `7070` free on your machine

## How to run

1. Save the compose file as `docker-compose.yml` in your project folder.

2. Start the containers:
   ```bash
   docker compose up -d
   ```

3. Check that both containers are running:
   ```bash
   docker ps
   ```
   You should see `hyperscale-kafka` and `hyperscale-kafka-ui` with status `Up`.

4. Open the Kafka UI in your browser:
   ```
   http://localhost:7070
   ```
   You should see a cluster named `hyperscale-cluster`. From here you can view topics, browse messages, and see consumer groups.

## How your services should connect

- **Services running on your host machine** (outside Docker, e.g. `npm run dev`):
  ```
  brokers: ["localhost:9092"]
  ```

- **Services running inside Docker** (in the same Docker network as Kafka):
  ```
  brokers: ["kafka:29092"]
  ```

## Stopping and restarting

Stop the containers (data is kept, since it's stored in a Docker volume):
```bash
docker compose down
```

Start them again later:
```bash
docker compose up -d
```

## Resetting everything (wipe all topics and messages)

If you want a completely fresh Kafka (no old topics, no old messages):
```bash
docker compose down -v
docker compose up -d
```
The `-v` flag deletes the `kafka-data` volume, so use this only when you're okay losing all topic data.

## Notes

- This is a **single-broker** setup, which is why replication settings (`KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR`, etc.) are all set to `1`. Don't remove these — Kafka defaults to expecting 3 brokers otherwise, and things will break.
- Port `9093` is used internally by Kafka for controller/metadata traffic — you don't need to connect to it from your services.
- You'll see a topic called `__consumer_offsets` appear automatically in the UI — this is normal, Kafka creates it itself to track consumer progress. Don't delete it.
