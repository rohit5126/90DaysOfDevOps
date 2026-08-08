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


