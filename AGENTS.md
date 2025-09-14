# Suricata Docker Environment Specification

## 1. Overview

This project provides a self-contained Docker environment for running and testing the Suricata Intrusion Detection System (IDS). The environment consists of three containers: a firewall container running Suricata, a client container to generate network traffic, and a server container to act as the destination for the traffic.

## 2. Architecture

The environment is built using Docker and Docker Compose. It consists of the following components:

*   **Client Container:** A lightweight container responsible for generating network traffic to the server.
*   **Firewall (Suricata) Container:** This container sits between the client and the server, inspecting all traffic that passes through it. It runs Suricata to monitor, log, and potentially block malicious traffic based on a defined ruleset.
*   **Server Container:** A container running a simple web server, which acts as the destination for the client's traffic.

### Network Configuration

A single Docker network will be created:

*   `viz-net`: A network connecting the `client`, `fw`, and `server` containers.

The `fw` container will monitor traffic on this network. Note that in this configuration, the `fw` container is not positioned inline between the `client` and `server`. Traffic between the `client` and `server` occurs directly over the `viz-net` bridge network. The `fw` container inspects this traffic in a passive (IDS) mode.

## 3. Container Specifications

### 3.1. `client-container`

*   **Base Image:** `alpine:latest`
*   **Software:** `curl`, `ping`, and other basic networking tools.
*   **Functionality:** Initiates traffic to the `server-container`.

### 3.2. `fw-container`

*   **Base Image:** `ubuntu:22.04`
*   **Software:** Suricata, networking tools.
*   **Configuration:**
    *   Suricata will be configured to run in IDS (Intrusion Detection System) mode, monitoring traffic on the `viz-net` network.
    *   It is configured to inspect traffic passively.
    *   A basic set of Suricata rules will be included to detect common threats.
    *   Logs from Suricata will be written to a volume for easy access.

### 3.3. `server-container`

*   **Base Image:** `nginx:alpine`
*   **Software:** Nginx web server.
*   **Functionality:** Serves a simple default webpage.

## 4. Project Structure

```
vizfire/
├── AGENTS.md
├── compose.yml
├── client/
│   └── Dockerfile
├── suricata/
│   ├── Dockerfile
│   ├── suricata.yaml
│   └── rules/
│       └── custom.rules
└── server/
    └── Dockerfile
```

## 5. Getting Started

1.  **Build the containers:** `docker-compose build`
2.  **Start the environment:** `docker-compose up -d`
3.  **Generate traffic:**
    *   Attach to the client container: `docker-compose exec client /bin/sh`
    *   From the client container, send a request to the server: `curl server`
4.  **View Suricata logs:**
    *   Logs will be available in the `suricata/logs/` directory on the host.
5.  **Stop the environment:** `docker-compose down`

## 6. Suricata Configuration (suricata.yaml) Explanation

This section explains the key configuration items in the `suricata.yaml` file.

### Main Configuration Items

1.  **`vars.address-groups.HOME_NET`**:
    *   **Role**: Defines the internal network to be protected. Many signatures use this variable to determine the direction of traffic (internal to external, external to internal).
    *   **Configuration**: Do not leave this as `any`. Always specify the CIDR of your actual internal network (e.g., `192.168.0.0/24`). This reduces false positives and significantly improves detection accuracy.

2.  **`rule-files`**:
    *   **Role**: Specifies the files containing the signatures (rules) that Suricata will use.
    *   **Configuration**: It is common to add open-source rule sets like [Emerging Threats](https://rules.emergingthreats.net/open/suricata/) in addition to the default rules, or to create and add your own rule files like `custom.rules`.

3.  **`outputs.eve-log`**:
    *   **Role**: Outputs almost all events detected and analyzed by Suricata (alerts, flow information, HTTP/DNS logs, etc.) in JSON format.
    *   **Configuration**: It is highly recommended to set `enabled: yes`. You can selectively enable the required log types (`alert`, `http`, `dns`, `flow`, etc.) in the `types` section. A modern approach is to collect this `eve.json` file with tools like Fluentd and forward it to a SIEM such as Elasticsearch or Splunk.

4.  **`af-packet`** (for Linux environments):
    *   **Role**: A mechanism for high-speed packet acquisition using Linux kernel features with zero-copy.
    *   **Configuration**: Specify the network interface you want to monitor (e.g., `eth0`, `enp0s3`) for `interface`. This is a critical setting for maximizing performance.

5.  **`host-os-policy`**:
    *   **Role**: By defining the host OS in the network, it improves the accuracy of rules that detect attacks targeting OS-specific vulnerabilities.
    *   **Configuration**: This is optional, but configuring it improves the detection capability against attacks targeting specific OSs.

Setting these items correctly according to your environment is the first step to operating Suricata effectively.