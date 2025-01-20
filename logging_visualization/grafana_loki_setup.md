# Grafana and Loki Setup Guide

This guide explains how to set up Grafana and Loki using Docker to enable visualization and monitoring of logs.

---

## Step 1: Install Docker

Follow the official Docker installation guide for your operating system. For Debian-based systems:

1. Visit [Docker Engine Installation Guide for Debian](https://docs.docker.com/engine/install/debian/).
2. Install Docker and start the service:
   ```bash
   sudo apt-get update
   sudo apt-get install -y docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

Verify that Docker is installed:
```bash
docker --version
```

---

## Step 2: Install Grafana via Docker

1. Pull the Grafana Docker image:
   ```bash
   sudo docker pull grafana/grafana:latest
   ```
2. Run the Grafana container:
   ```bash
   sudo docker run --name grafana -d -p 3000:3000 grafana/grafana:latest
   ```
   - `--name grafana`: Names the container.
   - `-d`: Runs the container in the background.
   - `-p 3000:3000`: Maps port 3000 on the host to port 3000 in the container for web access.

3. Verify that Grafana is running:
   ```bash
   sudo docker ps
   ```

Access Grafana via your web browser:
```plaintext
http://<external-ip-addr>:3000
```
The default username and password are `admin`. You will be prompted to change the password upon first login.

---

## Step 3: Install Loki via Docker

### Prepare the Loki Configuration File
Create a file named `loki-config.yaml` in your current directory with the following contents:
```yaml
auth_enabled: false
server:
  http_listen_port: 3100
  grpc_listen_port: 0
ingestion_rate_strategy: global
limits_config:
  enforce_metric_name: false
schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h
storage_config:
  boltdb_shipper:
    active_index_directory: /tmp/loki/boltdb-shipper-active
    cache_location: /tmp/loki/boltdb-shipper-cache
    shared_store: filesystem
  filesystem:
    directory: /tmp/loki/chunks
compactor:
  working_directory: /tmp/loki/boltdb-shipper-compactor
  shared_store: filesystem
```

### Run the Loki Container
1. Pull the Loki Docker image:
   ```bash
   sudo docker pull grafana/loki:3.0.0
   ```
2. Run the Loki container:
   ```bash
   sudo docker run --name loki -d \
     -v $(pwd)/loki-config.yaml:/mnt/config/loki-config.yaml \
     -p 3100:3100 \
     grafana/loki:3.0.0 \
     -config.file=/mnt/config/loki-config.yaml
   ```
   - The above command mounts the `loki-config.yaml` file and exposes port `3100` for Loki.

Restrict access to port `3100` by allowing connections only from Cowrie honeypot IPs. Refer to the `aws_setup.md` guide for firewall configurations.

---

## Step 4: Add Loki as a Data Source in Grafana

1. Log in to Grafana via the web interface:
   ```plaintext
   http://<external-ip-addr>:3000
   ```
2. Navigate to **Connections**:
   - Click the **hamburger menu** (three horizontal lines).
   - Go to **Connections** > **Add Data Source**.
3. Add Loki as a data source:
   - Select **Loki**.
   - Enter the Loki URL. If Grafana and Loki are on the same server, use the private NAT IP of the server:
     ```plaintext
     http://<nat-ip>:3100
     ```
4. Save and test the data source configuration.

---
