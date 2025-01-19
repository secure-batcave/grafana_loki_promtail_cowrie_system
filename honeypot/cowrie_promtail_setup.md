# Cowrie Honeypot and Promtail Setup Guide

## Step 1: VM Configuration and Cowrie Honeypot Installation

### 1. Update System and Install Dependencies
Run the following commands to update the system and install necessary dependencies for Cowrie:
```bash
sudo apt-get update
sudo apt-get install git python3-virtualenv libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind virtualenv
```

### 2. Create a User for Cowrie
Create a separate user to isolate Cowrie from the root account:
```bash
sudo adduser --disabled-password cowrie
sudo su cowrie
```

### 3. Clone and Set Up Cowrie under cowrie user
```bash
cd
git clone http://github.com/cowrie/cowrie
cd cowrie
pwd  # Verify the path is /home/cowrie/cowrie

# Create a virtual environment for Cowrie
virtualenv cowrie-env
source cowrie-env/bin/activate

# Update pip and install dependencies
pip install --upgrade pip
pip install --upgrade -r requirements.txt

# Configure Cowrie
cd etc/
cp cowrie.cfg.dist cowrie.cfg
vi cowrie.cfg
```
Edit the `cowrie.cfg` file to enable Telnet support:
```ini
# Enable Telnet support, disabled by default
enabled = true
```

### 4. Configure Firewall Rules
Return to the default 'ubuntu' user (ctrl+d) and set up iptables to redirect ports:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
```
Verify that your instance is listening to the correct ports:
```bash
ss -tan
```

### 5. Starting Cowrie
To start Cowrie:
```bash
sudo su cowrie
cd ~/cowrie
source cowrie-env/bin/activate
bin/cowrie start
```
Your Cowrie honeypot is now running with the default settings.

---

## Step 2: Promtail Installation and Configuration

### 1. Install Docker
Promtail is installed via Docker. If Docker is not installed, set it up:
```bash
sudo apt-get update
sudo apt-get install docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. Configure Promtail
Create a `promtail-config.yaml` file in your working directory:
```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://<loki-server-ip>:3100/loki/api/v1/push

scrape_configs:
- job_name: cowrie
  static_configs:
  - targets:
      - localhost
    labels:
      job: cowrie
      __path__: /var/log/cowrie.json

  pipeline_stages:
  - timestamp:
      source: timestamp
      format: RFC3339
```
Replace `<loki-server-ip>` with the IP of your Loki server.

### 3. Run Promtail Container
Run the Promtail container with the following command:
```bash
sudo docker run --name promtail -d \
  -v $(pwd):/mnt/config \
  -v /home/cowrie/cowrie/var/log/cowrie:/var/log \
  grafana/promtail:3.0.0 \
  -config.file=/mnt/config/promtail-config.yaml
```
Ensure that the user running Promtail has the necessary permissions to access Cowrie’s log files.

### 4. Grant Log Access
Allow the default `ubuntu` user access to Cowrie logs by adding it to the `cowrie` group:
```bash
sudo usermod -aG cowrie ubuntu
```
Log out and log back in for the changes to take effect.

---

## Testing the Setup
1. Verify that Cowrie is capturing activity by checking the logs:
   ```bash
   tail -f /home/cowrie/cowrie/var/log/cowrie/cowrie.json
   ```
2. Confirm Promtail is running and sending logs to Loki:
   ```bash
   docker logs promtail
   ```
3. In Loki, ensure logs from the Cowrie honeypot are visible under the job label `cowrie`.

With these steps, your Cowrie honeypot and Promtail setup is complete and ready for integration with Loki and Grafana for visualization.
