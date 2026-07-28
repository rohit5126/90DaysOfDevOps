# Node Exporter, cAdvisor, and Grafana Dashboards

Prometheus is running and you can query metrics. But right now it is only monitoring itself. In production, you need to monitor two critical things: the host machine (CPU, memory, disk, network) and the Docker containers running on it.

Today you add Node Exporter for host metrics, cAdvisor for container metrics, and set up Grafana to visualize everything in dashboards instead of raw PromQL.

## Task 1: Add Node Exporter for Host Metrics

**In docker compose file**
```
Service:
  node-exporter:
    image: prom/node-exporter:latest
    restart: always
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    commands:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring
```

**In prometheus.yml config file**

```
- job_name: "node-exporter"
    scrape_interval: 5s    # Overrides the global default
    metrics_path: "/metrics"
    static_configs:
      - targets: ["node-exporter:9100"]
```

**Why these volume mounts?**

/proc -- kernel and process information (CPU stats, memory info)
/sys -- hardware and driver details
/ -- filesystem usage (disk space)

**Queries to run**

```
# CPU: percentage of time spent idle (per core)
node_cpu_seconds_total{mode="idle"}

# Memory: total vs available
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk: filesystem usage percentage
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100

# Network: bytes received per second
rate(node_network_receive_bytes_total[5m])
```

---

## Task 2: Add cAdvisor for Container Metrics

**docker compose file**

```

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    restart: always
    privileged: true   #to provide access for /sys, /var/lib: ro
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /dev/disk/:/dev/disk:ro
    ports:
      - "8080:8080"
    networks:
      - monitoring
```

**Why these volume mounts?**

Docker socket (docker.sock) -- lets cAdvisor discover and query running containers
/sys -- kernel-level container stats (cgroups)
/var/lib/docker/ -- container filesystem information

**prometheus config file**

```
- job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
```

**Queries to run**

```
# CPU usage per container (in seconds)
rate(container_cpu_usage_seconds_total{name!=""}[5m])

# Memory usage per container
container_memory_usage_bytes{name!=""}

# Network received bytes per container
rate(container_network_receive_bytes_total{name!=""}[5m])

# Which container is using the most memory?
topk(3, container_memory_usage_bytes{name!=""})
```

---

## Task 3: Set Up Grafana

**docker compose file**

```
services:
  grafana:
    image: grafana/grafana:latest
    restart: always
    volumes: 
      - grafana_data:/var/lib/grafana
    ports:
      - "3000:3000"
    networks:
      - monitoring

```

**Add Prometheus as a datasource:**

Go to Connections > Data Sources > Add data source
Select Prometheus
Set URL to http://prometheus:9090 (use the container name, not localhost -- they are on the same Docker network)
Click Save & Test -- you should see "Successfully queried the Prometheus API"

---

## Task 4: Build Your First Dashboard

**Create a dashboard that shows the health of your system at a glance.**

**Go to Dashboards > New Dashboard > Add Visualization**

```
Select Prometheus as the datasource
Panel 1 -- CPU Usage (Gauge):

100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Visualization: Gauge
Title: "CPU Usage %"
Set thresholds: green < 60, yellow < 80, red >= 80
Panel 2 -- Memory Usage (Gauge):

(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
Visualization: Gauge
Title: "Memory Usage %"
Panel 3 -- Container CPU Usage (Time Series):

rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
Visualization: Time series
Title: "Container CPU Usage"
Legend: {{name}}
Panel 4 -- Container Memory Usage (Bar Chart):

container_memory_usage_bytes{name!=""} / 1024 / 1024
Visualization: Bar chart
Title: "Container Memory (MB)"
Legend: {{name}}
Panel 5 -- Disk Usage (Stat):

(1 - node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100
Visualization: Stat
Title: "Disk Usage %"
```

Save the dashboard as "DevOps Observability Overview".

---

## Task 5: Auto-Provision Datasources with YAML

**manually adding the datasource**

**grafana/provisioning/datasources/datasources.yml**

```
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
```

**Update the Grafana service in docker-compose.yml to mount the provisioning directory:**
```
  grafana:
    image: grafana/grafana-enterprise:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    restart: unless-stopped
```

**Why is provisioning datasources via YAML better than configuring them manually through the UI?**

Provisioning Grafana data sources via YAML configuration files is better than manual UI setup because it enables automation, version control, and consistency. It prevents configuration drift and allows seamless scaling across multiple environments.

---









