---
layout: default
title: "Alertmanager"
nav_order: 8
---

# Chapter 7: Alertmanager

In the previous chapter, [Blackbox Exporter](06_blackbox_exporter_.md), we learned how to monitor the availability of our applications. We know *if* something is down, but who gets notified? And how do we prevent getting spammed with alerts when there's a larger problem? That's where Alertmanager comes in!

Imagine your website goes down in the middle of the night. You don't want everyone on your team to get a phone call. You probably only want the on-call engineer to get notified. And you definitely don't want to get a separate alert for *every single failing check*. Alertmanager helps you manage these alerts!

Alertmanager is like a central notification system for Prometheus. When Prometheus detects a problem (based on rules we'll define later in [Alerting Rules (rules.yml)](09_alerting_rules__rules_yml__.md)), it sends an alert to Alertmanager. Alertmanager then:

1.  **Groups** similar alerts.
2.  **Deduplicates** redundant alerts.
3.  **Routes** the alerts to the correct receiver (e.g., Slack, email).
4.  **Suppresses** alerts to prevent notification floods.

Think of it as a smart alarm system that filters and prioritizes notifications, ensuring the right people are notified at the right time, and preventing alert fatigue.

## Key Concepts of Alertmanager

Let's break down the core components of Alertmanager:

*   **Alerts:** These are notifications sent from Prometheus when a defined condition is met (e.g., CPU usage is too high, a service is down).
*   **Receivers:** These define where the alerts should be sent (e.g., Slack channel, email address).
*   **Routes:** These determine *how* alerts are routed to receivers. You can define rules based on alert labels (e.g., severity, service name) to send different alerts to different receivers.
*   **Grouping:** This groups similar alerts together to reduce noise.
*   **Inhibition:** This suppresses certain alerts based on other alerts. For instance, if a critical system is down, you might want to suppress alerts about individual components failing within that system.

## Getting Started with Alertmanager

In our `docker-compose.yaml` file, we already have an Alertmanager service defined:

```yaml
  alertmanager:
    image: prom/alertmanager:v0.23.0
    container_name: alertmanager
    restart: unless-stopped
    ports:
      - "9093:9093"
    networks:
      - prom_net
    volumes:
      - "./am_config:/config"
      - "alertmanager_data:/data"
    command: --config.file=/config/alertmanager.yml --log.level=debug
```

Let's break down what this means:

*   `image: prom/alertmanager:v0.23.0`: This specifies the Docker image to use for Alertmanager.
*   `container_name: alertmanager`: This gives the container a name.
*   `restart: unless-stopped`: Ensures Alertmanager restarts if it crashes.
*   `ports: - "9093:9093"`: This maps port 9093 on your host machine to port 9093 on the container. You'll use this port to access the Alertmanager web interface.
*   `networks: - prom_net`: This attaches the Alertmanager container to the `prom_net` network.
*   `volumes:`:
    *   `- "./am_config:/config"`: This mounts a directory `./am_config` on your host machine to the `/config` directory inside the container. This is where Alertmanager expects to find its configuration file (`alertmanager.yml`).
    *   `- "alertmanager_data:/data"`: This creates a volume to persist Alertmanager's data, like silences.
*   `command: --config.file=/config/alertmanager.yml --log.level=debug`: This tells Alertmanager where to find its configuration file and sets the log level to debug.

## Configuring Alertmanager

The heart of Alertmanager is its configuration file, `alertmanager.yml`. This file defines the receivers, routes, grouping, and inhibition rules.

Here's a simplified version of our `am_config/alertmanager.yml` file:

```yaml
global:
  resolve_timeout: 5m

receivers:
- name: "null"
- name: 'application_status'
  slack_configs:
    - api_url: <api-url> # Put Slack integrated webhook URL here"
      channel: '<channel-name>' # the channel name should have the '#' character with no space between.
      text: "<!channel> \nDescription: {{ .CommonAnnotations.description }}"
      title: "{{ .CommonAnnotations.summary }}"
      send_resolved: true

route:
  group_by:
    - severity
  receiver: "null" #Default receiver

  routes:
  - match:
      alertname: BackendDown
    receiver: 'application_status'
    continue: true
  - match:
      alertname: FrontendDown
    receiver: 'application_status'
    continue: true
```

**Explanation:**

*   `global:`: Defines global settings, like the `resolve_timeout`.
*   `receivers:`: Defines the notification endpoints.
    *   `name: "null"`: A special receiver that discards alerts. This is useful as the default receiver.
    *   `name: 'application_status'`: A receiver that sends alerts to a Slack channel. **You'll need to replace `<api-url>` with your Slack webhook URL and `<channel-name>` with your Slack channel name.**  To get a slack webhook URL, you'll need to set up a Slack App.
*   `route:`: Defines how alerts are routed to receivers.
    *   `group_by: - severity`: Groups alerts by severity (e.g., critical, warning, info).
    *   `receiver: "null"`: Sets the default receiver to "null," meaning alerts will be discarded *unless* they match a specific route.
    * `routes`: Defines specific routing rules
      * `match`: Route `BackendDown` alerts to `application_status` receiver
      * `match`: Route `FrontendDown` alerts to `application_status` receiver

**Important:** You need to set up your own Slack webhook and channel for the `application_status` receiver to work.

## Connecting Prometheus to Alertmanager

We need to tell Prometheus where to send its alerts. This is done in the `alerting` section of the `prom_config/prometheus.yml` file:

```yaml
alerting:
  alertmanagers:
    - static_configs:
      - targets:
        - alertmanager:9093
```

This tells Prometheus to send alerts to the `alertmanager` container on port 9093.

## Accessing the Alertmanager Web Interface

If you've followed the previous chapters and run `docker-compose up`, you should be able to access the Alertmanager web interface by opening your browser and going to `http://localhost:9093`.

You'll see a page that shows the currently active and silenced alerts.

## Testing Alertmanager

To test Alertmanager, we need to trigger an alert from Prometheus. We'll cover how to define alerting rules in detail in [Alerting Rules (rules.yml)](09_alerting_rules__rules_yml__.md), but for now, let's assume we have a simple rule that triggers when our backend is down. This would trigger the `BackendDown` alert that is routed to the `application_status` receiver.

If everything is configured correctly, you should see an alert in the Alertmanager web interface and receive a notification in your Slack channel (if you configured the `application_status` receiver).

## Internal Implementation

Let's see what happens behind the scenes when Prometheus sends an alert to Alertmanager.

```mermaid
sequenceDiagram
    participant Prometheus
    participant Alertmanager
    participant Receiver (e.g., Slack)

    Prometheus->>Alertmanager: Send Alert
    Alertmanager->>Alertmanager: Group & Deduplicate
    Alertmanager->>Alertmanager: Route Alert
    Alertmanager->>Receiver: Send Notification
```

1.  Prometheus detects a problem based on its alerting rules and sends an alert to Alertmanager.
2.  Alertmanager groups and deduplicates similar alerts.
3.  Alertmanager routes the alert to the appropriate receiver based on the routing rules.
4.  Alertmanager sends a notification to the receiver (e.g., Slack, email).

The Alertmanager code is primarily written in Go. Key parts of the codebase handle:

*   **Configuration Parsing:** Reads the `alertmanager.yml` file and parses the configuration.
*   **Alert Processing:** Handles incoming alerts, groups them, and deduplicates them.
*   **Routing:** Matches alerts to receivers based on routing rules.
*   **Notification Sending:** Sends notifications to configured receivers.

Here's a simplified, hypothetical example of how Alertmanager might route an alert:

```go
// Not actual Alertmanager code
package main

import "fmt"

// Simplified alert struct
type Alert struct {
	Alertname string
}

// Hypothetical routing function
func routeAlert(alert Alert) string {
	// In a real application, this would use the alertmanager.yml config
	// For this example, we'll hardcode the routing logic.

	if alert.Alertname == "BackendDown" {
		return "slack" // Send to Slack
	} else if alert.Alertname == "FrontendDown" {
		return "email" // Send to Email
	} else {
		return "null" // Discard alert
	}
}

func main() {
	backendDownAlert := Alert{Alertname: "BackendDown"}
	route := routeAlert(backendDownAlert)
	fmt.Printf("Routing BackendDown alert to: %s\n", route)

	frontendDownAlert := Alert{Alertname: "FrontendDown"}
	route = routeAlert(frontendDownAlert)
	fmt.Printf("Routing FrontendDown alert to: %s\n", route)
}
```

**Explanation:**

This simplified program *simulates how Alertmanager might route alerts based on their `Alertname`*. A real Alertmanager implementation would use more complex routing logic based on labels and matchers.

## Conclusion

Alertmanager is a crucial component of a robust monitoring system. It allows you to effectively manage alerts, prevent notification floods, and ensure that the right people are notified when problems occur. We've covered the basics of Alertmanager, how to configure it, and how it integrates with Prometheus. In the next chapter, [Prometheus Configuration (prometheus.yml)](08_prometheus_configuration__prometheus_yml__.md), we'll dive deeper into the `prometheus.yml` file and explore more advanced configuration options.


