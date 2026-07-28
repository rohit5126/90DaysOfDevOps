# Log Management with Loki and Promtail

Metrics tell you what is broken. Logs tell you why. Yesterday you built the metrics pipeline with Prometheus, Node Exporter, cAdvisor, and Grafana. Today you add the second pillar of observability -- logs.

You will set up Grafana Loki (a log aggregation system built by the Grafana team) and Promtail (the agent that ships logs to Loki).

## Task 1: Understand the Logging Pipeline

```
[Docker Containers]
       |
       | (write JSON logs to /var/lib/docker/containers/)
       v
  [Promtail]
       |
       | (reads log files, adds labels, pushes to Loki)
       v
    [Loki]
       |
       | (stores logs, indexes by labels)
       v
   [Grafana]
       |
       | (queries Loki with LogQL, displays logs)
       v
   [You]
```

**Key differences from the ELK stack:**

Loki does not index the full text of logs -- it only indexes labels (like container name, job, filename)
This makes Loki much cheaper to run and simpler to operate
Think of it as "Prometheus, but for logs" -- same label-based approach


**Why does Loki only index labels instead of full text? What is the trade-off?**

Grafana Loki only indexes labels to dramatically lower operational costs, memory usage, and storage overhead. It trades instant search speeds across massive datasets for a significantly cheaper, more scalable architecture modeled after Prometheus

---

## Task 2: Add Loki to the Stack

**loki configuration file**

```
auth_enabled: false

server:
  http_listen_port: 3100

common:
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory
  replication_factor: 1
  path_prefix: /loki

schema_config:
  configs:
    - from: "2020-10-24"
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /loki/chunks

```

**What this config does:**

**auth_enabled: false --** single-tenant mode, no authentication needed
**store: tsdb --** uses Loki's time-series database for indexing
**object_store: filesystem --** stores log chunks on local disk
**replication_factor: 1 --** single instance, no replication (fine for learning)

**docker compose file**

```

services:
  loki:
    image: grafana/loki:latest
    restart: always
    volumes:
      - ./loki/loki-config.yaml:/etc/loki/config.yaml
      - loki_data:/loki
    commands:
      - config.file=/etc/loki/config.yaml
    ports:
      - "3100:3100"
    networks:
      - monitoring

```

---

## Task 3: Add Promtail to Collect Container Logs

**promtail config file**

```
server:
  http_listen_port: 9080

positions:
  filename: /tmp/postions.yml

clients:
  - url: http://loki:3100/loki/api/v1/push    # name of the service in docker compose loki so its loki:3100

scrape_configs:
  - job_name: docker
    static_configs:
      - targets:  
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*-json.log
    pipeline_stages:
      - docker: {}
```

**What this config does:**

**positions --** tracks which log lines have already been shipped (like a bookmark)
**clients --** where to send logs (Loki endpoint)
**__path__ --** the glob pattern to find Docker JSON log files on the host
**pipeline_stages: docker: {} --** parses the Docker JSON log format and extracts timestamp, stream (stdout/stderr), and the log message

**docker compose file**

```
services:
  promtail:
    image: grafana/promtail:latest
    restart: always
    volumes:
      - /var/log:/var/log:ro
      - ./promtail/promtail-config.yml:/etc/promtail/config.yml:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    commands:
      - config.file=/etc/promtail/config.yml
    depends_on:
      - loki
    networks:
      - monitoring

```

**Why these volume mounts?**

**/var/lib/docker/containers --** where Docker stores container log files (read-only)
**/var/run/docker.sock --** lets Promtail discover container metadata (names, labels)

---

## Task 5: Query Logs with LogQL
