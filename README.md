# Honeypot Monitoring and Visualization System

## Overview

This repository provides a fundamental guide to setting up a honeypot monitoring and visualization system. By combining a Cowrie honeypot with logging and visualization tools such as Grafana, Loki, and Promtail, this project demonstrates an end-to-end solution for collecting and analyzing suspicious network traffic.

This guide utilizes AWS free tier resources, making the setup completely free. All that is required is a credit card to verify your account, and Google Authenticator (or similar) for MFA.

## Features
- **Honeypot Setup:** Deploy and configure a Cowrie honeypot to capture SSH/Telnet activity.
- **Centralized Logging:** Use Loki and Promtail to aggregate and store logs from the honeypot.
- **Data Visualization:** Leverage Grafana to visualize trends and insights from collected logs.
- **Cloud Integration:** Utilize AWS for hosting.

## Technologies Used
- **Cowrie Honeypot**: Simulated SSH and Telnet server to collect attack data.
- **Grafana**: Visualization and monitoring tool.
- **Loki**: Log aggregation system.
- **Promtail**: Log forwarding agent.
- **AWS**: Cloud hosting platform.

## Repository Structure
```
/honeypot-monitoring-system
├── /honeypot
│   ├── cowrie_promtail_setup.md    # Detailed honeypot setup guide
│   ├── cowrie_configs.md           # Example configuration files
├── /logging_visualization
│   ├── grafana_loki_setup.md       # Grafana and Loki setup guide
├── /server
│   ├── aws_setup.md                # AWS account creation and setup
├── README.md                       # Overview of the project
```

## Getting Started

Follow the steps below to set up the honeypot monitoring system:

1. **AWS Setup**
   - Create AWS accounts and configure the necessary resources. ([Guide](./server/aws_setup.md))

2. **Honeypot Setup**
   - Deploy and configure the Cowrie honeypot to simulate a vulnerable environment. Promtail ships logs to Loki. ([Guide](./honeypot/cowrie_promtail_setup.md))
   - Further configure the Cowrie honeypot. ([Guide](./honeypot/cowrie_configs.md))

3. **Logging and Visualization**
   - Install and configure Grafana and Loki for log aggregation and visualization. ([Guide](./logging_visualization/grafana_loki_setup.md))

## Goals and Applications

The primary goal of this project is to:
- Collect and analyze attack data from a simulated honeypot environment.
- Provide actionable insights through visualized log data.
- Demonstrate a cohesive integration of cybersecurity, logging, and visualization technologies.

### Practical Applications
- **Threat Analysis**: Identify patterns in attack behavior.
- **Security Monitoring**: Monitor real-time attacks and potential threats.
- **Skill Building**: Gain hands-on experience in cybersecurity, cloud hosting, and visualization.