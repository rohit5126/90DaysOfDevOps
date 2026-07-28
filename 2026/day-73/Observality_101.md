# Introduction to Observability and Prometheus

## Task 1: Understand Observability

Observability is the measure of how well you can understand a system's internal state by looking at its external outputs, primarily using metrics, logs, and traces. Traditional monitoring tells you that a system is broken, while observability helps you figure out why it is broken

## The three pillars of observability:

**Metrics --** numerical measurements over time (CPU usage, request count, error rate). Tools: Prometheus, Datadog, CloudWatch

**Logs --** timestamped text records of events (application output, error messages). Tools: Loki, ELK Stack, Fluentd

**Traces --** the journey of a single request across multiple services. Tools: OpenTelemetry, Jaeger, Zipkin.

## Why do DevOps engineers need all three?

DevOps engineers need all three pillars because modern applications are highly complex and distributed. Relying on just one or two pillars leaves critical blind spots that prolong system outages.

```
[bankapp] --> metrics --> [Prometheus] --> [Grafana Dashboards]
[bankapp] --> logs    --> [Promtail]   --> [Loki] --> [Grafana]
[bankapp] --> traces  --> [OTEL Collector] --> [Grafana/Debug]
[Host]     --> metrics --> [Node Exporter] --> [Prometheus]
[Docker containers]   --> metrics --> [cAdvisor] --> [Prometheus]

```

---

## Task 2: Set Up Prometheus with Docker

**create a prometheus.yml**

```
global:
  scrape_interval: 15s     # How often to scrape targets (default: 1m)
  evaluation_interval: 15s # How often to evaluate rules (default: 1m)
  scrape_timeout: 10s

scrape_configs:
  # Job 1: Monitor Prometheus itself
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]
```

**docker-compose.yml file**

```
services:
  prometheus:
    image: prom/prometheus:latest
    restart: always
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    commands:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - monitoring
```

## Task 3: Understand Prometheus Concepts

Scrape targets -- endpoints that Prometheus pulls metrics from at regular intervals (pull-based model)
Metrics types:
Counter -- only goes up (total requests served, total errors)
Gauge -- goes up and down (current CPU usage, memory in use, active connections)
Histogram -- distribution of values in buckets (request duration: how many took <100ms, <500ms, <1s)
Summary -- similar to histogram but calculates percentiles on the client side
Labels -- key-value pairs that add dimensions to metrics (e.g., http_requests_total{method="GET", status="200"})
Time series -- a unique combination of metric name + labels
Go to the Prometheus UI graph page (http://localhost:9090/graph) and run these queries:

**How many metrics is Prometheus collecting about itself?**
count({__name__=~".+"})

**How much memory is Prometheus using?**
process_resident_memory_bytes

**Total HTTP requests to the Prometheus server**
prometheus_http_requests_total

**Break it down by handler**
prometheus_http_requests_total{handler="/api/v1/query"}

**Document: What is the difference between a counter and a gauge? Give one real-world example of each.**

A counter tracks a cumulative, total number of events that only goes up, while a gauge shows a current value that can freely go up and down.

## Task 4: Learn PromQL Basics

**PromQL (Prometheus Query Language) is how you ask questions about your metrics. Run these queries in the Prometheus UI:**

**Instant vector -- current value of a metric:**
```
up
This returns 1 (up) or 0 (down) for each scrape target.
```

**Range vector -- values over a time window:**
```
prometheus_http_requests_total[5m]
Returns all values from the last 5 minutes.
```

**Rate -- per-second rate of a counter over a time window:**
```
rate(prometheus_http_requests_total[5m])
This is the most common function you will use. Counters always go up -- rate() converts them to a useful per-second speed.
```

**Aggregation -- sum across all label combinations:**
```
sum(rate(prometheus_http_requests_total[5m]))
```

**Filter by label:**
```
prometheus_http_requests_total{code="200"}
prometheus_http_requests_total{code!="200"}
```

**Arithmetic:**
```
process_resident_memory_bytes / 1024 / 1024
This converts bytes to megabytes.
```

**Top-K:**
```
topk(5, prometheus_http_requests_total)
```

**Try this exercise: Write a PromQL query that shows the per-second rate of non-200 HTTP requests to Prometheus over the last 5 minutes. (Hint: use rate() with a label filter on code!="200")**
```
rate(prometheus_http_requests_total{code="200"}[5m])
```

---

## Task 5: Add a Sample Application as a Scrape Target

**ran the AI bank application using docker compose and ran the prometheus conatiner in same network.**

```
networks:
  monitoring:
    external: true
    name: ai-bankapp-devops_bankapp-net
```

```
- job_name: "bankapp"
    scrape_interval: 10s
    metrics_path: '/metrics' 
    follow_redirects: true
    static_configs:
      - targets: ["bankapp:8080"]
```

**currently it is not working because /metrics is not defined in the app**

Node Exporter, cAdvisor, and OTEL Collector act as metric exporters for systems that do not have built-in Prometheus support.

---

## Task 6: Explore Data Retention and Storage

**Check how much disk space Prometheus is using:**
```
docker exec prometheus du -sh /prometheus
```

**Prometheus stores data in a local time-series database (TSDB). Default retention is 15 days. You can change it:**
```
command:
  - '--config.file=/etc/prometheus/prometheus.yml'
  - '--storage.tsdb.retention.time=30d'
  - '--storage.tsdb.retention.size=1GB'
```

**What happens when retention is exceeded? Why is a volume mount important for Prometheus data?**

When data retention limits (time or size) are exceeded, Prometheus automatically deletes the oldest time-series data blocks from its local storage. A volume mount is vital because it maps container data to external host or cloud storage, preventing the loss of monitoring metrics and history when a container restarts
