---
layout: default
title: "Home"
nav_order: 1
---

# Tutorial: container-monitoring

This project uses a **monitoring system** to keep track of the health and performance of a multi-container application.  It uses *Prometheus* to collect metrics, *Grafana* to visualize them, and *Alertmanager* to send notifications when something goes wrong.  *Docker Compose* makes it easy to run all these components together.


**Source Repository:** [https://github.com/samin-irtiza/container-monitoring](https://github.com/samin-irtiza/container-monitoring)

```mermaid
flowchart TD
    A0["Docker Compose
"]
    A1["Prometheus
"]
    A2["Alertmanager
"]
    A3["Node Exporter
"]
    A4["Blackbox Exporter
"]
    A5["Grafana
"]
    A6["PromQL (Prometheus Query Language)
"]
    A7["Prometheus Configuration (prometheus.yml)
"]
    A8["Alerting Rules (rules.yml)
"]
    A9["cAdvisor
"]
    A0 -- "Orchestrates" --> A1
    A0 -- "Orchestrates" --> A2
    A0 -- "Orchestrates" --> A3
    A0 -- "Orchestrates" --> A4
    A0 -- "Orchestrates" --> A5
    A0 -- "Orchestrates" --> A9
    A1 -- "Scrapes metrics" --> A3
    A1 -- "Scrapes metrics" --> A4
    A1 -- "Scrapes metrics" --> A9
    A1 -- "Sends alerts" --> A2
    A1 -- "Uses for queries" --> A6
    A1 -- "Reads configuration" --> A7
    A1 -- "Reads rules" --> A8
    A5 -- "Visualizes metrics" --> A1
    A8 -- "Uses in expressions" --> A6
    A4 -- "Exposes metrics" --> A1
```

## Chapters

1. [Docker Compose](01_docker_compose_.md)
2. [Prometheus](02_prometheus_.md)
3. [Grafana](03_grafana_.md)
4. [Node Exporter](04_node_exporter_.md)
5. [cAdvisor](05_cadvisor_.md)
6. [Blackbox Exporter](06_blackbox_exporter_.md)
7. [Alertmanager](07_alertmanager_.md)
8. [Prometheus Configuration (prometheus.yml)](08_prometheus_configuration__prometheus_yml__.md)
9. [Alerting Rules (rules.yml)](09_alerting_rules__rules_yml__.md)
10. [PromQL (Prometheus Query Language)](10_promql__prometheus_query_language__.md)


