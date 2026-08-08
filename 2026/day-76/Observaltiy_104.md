# Day 76 -- OpenTelemetry and Alerting

You have metrics (Prometheus) and logs (Loki). Today you add the third pillar -- traces -- using OpenTelemetry, the industry-standard framework for collecting telemetry data. Then you set up alerting so your system notifies you when something goes wrong, instead of you staring at dashboards all day.

## Task 1: Understand OpenTelemetry

OpenTelemetry is an open source framework which helps applications and systems generate, collect, and export telemetry data—specifically **traces, metrics, and logs**.

**Core Components**

**API & SDK -** it is used to write instrumentation code to generate data

**OTLP (OpenTelemetry Protocol):** it is used to safely transmit telemetry data over HTTP or gRPC.

**The Collector:** it is a service that receives, processes, filters, and exports data to different monitoring backends

#### What is the OTEL Collector?

The OpenTelemetry (OTel) Collector is a high-performance, vendor-agnostic proxy component.

**Three components in the pipeline:**

Receivers -- accept data (OTLP, Prometheus, Jaeger formats)

Processors -- transform data (batching, filtering, sampling)

Exporters -- send data to backends (Prometheus, debug console, Jaeger)

#### What are distributed traces?

an observability technique that tracks the complete path, timing, and status of a request as it flows across multiple microservices, databases, and external network systems.

A trace tracks a single request as it travels through multiple services                                         
Each step in the trace is called a span                                                        
Spans have: trace ID, span ID, parent span ID, start time, duration, attributes                                                      
Example: User request -> API Gateway (span 1) -> Auth Service (span 2) -> Database (span 3)                                   

---

## Task 2: Add the OpenTelemetry Collector

Create a collector config /otel-collector-config.yml:

```
receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      batch: {}
      memory_limiter:
        check_interval: 5s
        limit_mib: 400
        spike_limit_mib: 100
      resource:
        attributes:
          - key: k8s.cluster.name
            value: bankapp-eks
            action: upsert

    exporters:
      otlp/tempo:
        endpoint: tempo.monitoring.svc.cluster.local:4317
        tls:
          insecure: true
      prometheusremotewrite:
        endpoint: http://monitoring-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090/api/v1/write
      debug:
        verbosity: normal

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, resource, batch]
          exporters: [otlp/tempo, debug]
        metrics:
          receivers: [otlp]
          processors: [memory_limiter, resource, batch]
          exporters: [prometheusremotewrite]
```

**What this config does:**

**Receivers:** Accepts OTLP data via gRPC (4317) and HTTP (4318)                                                                                  
**Processors:** Batches data before exporting (reduces overhead)                                                                     
**Exporters:**                                                                                                
* Metrics go to a Prometheus-compatible endpoint on port 8889 (Prometheus scrapes this)
* Traces and logs go to debug output (console) -- in production you would send these to Jaeger or Tempo


**Add the collector to your docker-compose.yml:**

```
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    container_name: otel-collector
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
      - "8889:8889"   # Prometheus exporter
    volumes:
      - ./otel-collector/otel-collector-config.yml:/etc/otelcol-contrib/config.yaml
    restart: unless-stopped

```

**Add the OTEL Collector as a Prometheus scrape target in prometheus.yml:**

```
- job_name: "otel-collector"
    static_configs:
      - targets: ["otel-collector:8889"]

```

Verify it is running

`docker logs otel-collector 2>&1 | tail -5`

Check Prometheus Targets -- you should now see otel-collector

---

## Task 4: Set Up Prometheus Alerting Rules

Create an alerting rules file alert-rules.yml:

```
groups:
  - name: system-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage has been above 80% for more than 2 minutes. Current value: {{ $value }}%"

      - alert: HighMemoryUsage
        expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 85
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 85%. Current value: {{ $value }}%"

      - alert: ContainerDown
        expr: absent(container_last_seen{name="notes-app"})
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container is down"
          description: "The notes-app container has not been seen for over 1 minute"

      - alert: TargetDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Scrape target is down"
          description: "{{ $labels.job }} target {{ $labels.instance }} is unreachable"

      - alert: HighDiskUsage
        expr: (1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space running low"
          description: "Root filesystem usage is above 90%. Current value: {{ $value }}%"

```

Update prometheus.yml to load the rules:

```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:                          #add the rules for adding rule file
  - /etc/prometheus/alert-rules.yml   
```

Mount the rules file as volume in docker-compose.yml under the Prometheus service:

```
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert-rules.yml:/etc/prometheus/alert-rules.yml
```

Check the rules in the Prometheus UI: go to Status > Rules. You should see all five alert rules listed.

Go to Alerts -- they should be in inactive state (green). If any condition is true, the alert moves to pending, then firing after the for duration.

---

## Task 5: Set Up Grafana Alerts

Grafana can also evaluate alerts and send notifications to Slack, email, PagerDuty, and more.

```
Create a contact point:

Go to Alerting > Contact points > Add contact point
Name: "DevOps Team"
Integration: Choose email (or Slack webhook if you have one)
For email: just enter your email address
Save
Create an alert rule in Grafana:

Go to Alerting > Alert rules > New alert rule
Name: "High Container Memory"
Query: container_memory_usage_bytes{name="notes-app"} / 1024 / 1024
Condition: IS ABOVE 100 (fire if container uses more than 100MB)
Evaluation: every 1m, for 2m
Add label: severity = warning
Link to the "DevOps Team" contact point
Save
Create a notification policy:

Go to Alerting > Notification policies
Set the default contact point to "DevOps Team"
Add a nested policy: match label severity=critical -> route to a different contact point (or the same one with different settings)
View alert state:

Go to Alerting > Alert rules
You should see your rule in Normal, Pending, or Firing state
```

Prometheus alerts are evaluated at the data source level using PromQL via Prometheus servers and routed using a standalone Alertmanager. Grafana alerts are evaluated inside the Grafana platform itself and can use any data source, including logs, SQL databases, and cloud services, alongside Prometheus

---

## Task 6: Review the Full Stack Architecture

Services running:


| Service | Port | Purpose |
|---------|------|---------|
| Prometheus | 9090 | Metrics storage and querying |
| Node Exporter | 9100 | Host system metrics |
| cAdvisor | 8080 | Container metrics |
| Grafana | 3000 | Visualization and alerting |
| Loki | 3100 | Log storage |
| Promtail | 9080 | Log collection agent |
| OTEL Collector | 4317/4318/8889 | Telemetry collection |


