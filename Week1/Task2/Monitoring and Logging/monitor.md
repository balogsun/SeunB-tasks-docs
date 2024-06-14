## Introduction
This covers the steps to install and configure Prometheus for monitoring system metrics, set up Grafana for data visualization, create custom metrics in a Python application, configure alerting rules, and integrate Alertmanager for notifications. Additionally, the guide includes steps to set up SMTP for email notifications and create informative dashboards in Grafana.
## Install Prometheus

### Create System User or System Account

```sh
sudo useradd --system --no-create-home --shell /bin/false prometheus
```

### Download and Install Prometheus

```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
sudo mkdir -p /data /etc/prometheus
cd prometheus-2.47.1.linux-amd64/
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
cd ..
rm -rf prometheus-2.47.1.linux-amd64.tar.gz
prometheus --version
```

### Create Prometheus Systemd Service

```sh
sudo vim /etc/systemd/system/prometheus.service
```

Add the following content:

```ini
[Unit]
Description=prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

### Enable and Start Prometheus Service

```sh
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
journalctl -u prometheus -f --no-pager [troubleshoot]
```

Check <http://localhostIP:9090>. If you go to targets, you should see only one — Prometheus target. It scrapes itself every 15 seconds by default.

<img width="566" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/03ada490-844f-4acc-b5e8-9db6dc74ec37">

---

# Install NodeExporter

Node Exporter is a Prometheus exporter for hardware and OS metrics.

### Create System User

```sh
sudo useradd --system --no-create-home --shell /bin/false node_exporter
```

### Download and Install NodeExporter

```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
node_exporter --version
node_exporter --help
```

### Create NodeExporter Systemd Service

```sh
sudo vim /etc/systemd/system/node_exporter.service
```

Add the following content:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
```

### Enable and Start NodeExporter Service

```sh
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
journalctl -u node_exporter -f --no-pager [troubleshoot]
```

Open a browser and navigate to `http://localhost:9100/metrics`. You should see various system metrics being exposed.
<img width="527" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/c13ddb3f-be40-44a0-99f5-f04067a2233d">

### Add NodeExporter as a Target for Prometheus

```sh
sudo vim /etc/prometheus/prometheus.yml
```

Add the following job configuration:

```yaml
- job_name: node_export
    static_configs:
      - targets: ["localhost:9100"]
```

### Validate Prometheus Configuration

```sh
promtool check config /etc/prometheus/prometheus.yml
```

Check the output:

```sh
Checking /etc/prometheus/prometheus.yml
SUCCESS: /etc/prometheus/prometheus.yml is valid prometheus config file syntax
```

### Reload Prometheus Configuration

```sh
curl -X POST http://localhost:9090/-/reload
```

Or save the file and restart Prometheus:

```sh
sudo systemctl restart prometheus
```

## Check the Targets Section

1. Check `http://172.31.206.231:9090/targets`.

2. Open the Prometheus web UI (`http://localhost:9090`), go to `Status` > `Targets`, and ensure `node_exporter` is listed and UP.

<img width="664" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/ae96115d-764f-483e-b6f8-2f23b20384d3">

## Install Grafana

1. Install dependencies:

    ```bash
    sudo apt-get install -y apt-transport-https software-properties-common
    ```

2. Add the GPG key:

    ```bash
    wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
    ```

3. Add the repository for stable releases:

    ```bash
    echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```

4. Update and install Grafana:

    ```bash
    sudo apt-get update
    sudo apt-get -y install grafana
    ```

5. Enable and start the Grafana service:

    ```bash
    sudo systemctl enable grafana-server
    sudo systemctl start grafana-server
    sudo systemctl status grafana-server
    ```

6. Log in to Grafana at `http://20.63.110.74:3000` using default credentials:
    - Username: `admin`
    - Password: `admin`
      
<img width="755" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/31056672-073f-4543-9ccc-3cdc154b3911">

## Visualize Metrics in Grafana

1. Add a data source:
    - Click `Add data source` and select `Prometheus`.
    - OR Open the 3-line menu icon, select `connections` then `data sources`.
    - For the Prometheus server URL, enter `http://localhost:9090` OR `http://20.63.110.74:31638` if running from pods.
    - Click on `Save and Test`.

## Create Custom Metrics in Your Application

### Install the Prometheus Client Library for Python

```bash
pip install prometheus_client
```

### Create a Custom Metric in Your Application

Here’s an example Python script to create and expose custom metrics:

```python
from prometheus_client import start_http_server, Summary, Counter
import random
import time

# Create a metric to track time spent and requests made.
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')
REQUEST_COUNT = Counter('request_count', 'Number of requests processed')

@REQUEST_TIME.time()
def process_request(t):
    """A dummy function that takes some time."""
    time.sleep(t)
    REQUEST_COUNT.inc()

if __name__ == '__main__':
    # Start up the server to expose the metrics.
    start_http_server(8000)
    # Generate some requests.
    while True:
        process_request(random.random())
```

### Run Your Application in the Background and Redirect Output

```bash
nohup python3 custom_script.py > custom_script_output.log 2>&1 &
```

### Verify the Process

```bash
ps aux | grep custom_script.py
```

### Verify Custom Metrics

Open a browser and navigate to `http://localhost:8000/metrics`. Your custom metrics should be listed.

<img width="542" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/77b6e5be-4609-4cc5-9b6f-4ad099a00b79">

### Configure Prometheus to Scrape Your Custom Metrics

Edit `prometheus.yml` to include:

```yaml
scrape_configs:
  - job_name: 'custom_metrics'
    static_configs:
      - targets: ['localhost:8000']
```

### Restart Prometheus

```bash
sudo systemctl restart prometheus
```

### Verify Prometheus is Scraping Your Custom Metrics

Open the Prometheus web UI, go to `Status` > `Targets`, and ensure `custom_metrics` is listed and UP.

<img width="680" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/b22db34a-eb15-4166-80ae-c2d2bf8655ba">

## Create an Alert Rules File (`alert.rules`)

```bash
cd /etc/prometheus$
sudo vi alert.rules
```

Add the following rules:

```yaml
groups:
  - name: node_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage has been above 80% for more than 1 minute."

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 10
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High Memory usage detected"
          description: "Memory usage has been above 10% for more than 1 minute."
```

### Restart Prometheus

```bash
sudo systemctl restart prometheus
```

## Configure Alertmanager for Notifications

### Install Alertmanager

1. Create a user and group for Alertmanager:

    ```bash
    sudo useradd -M -r -s /bin/false alertmanager
    ```

2. Download and install the Alertmanager binaries:

    ```bash
    wget https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz
    tar xvfz alertmanager-0.20.0.linux-amd64.tar.gz
    sudo cp alertmanager-0.20.0.linux-amd64/alertmanager /usr/local/bin/
    sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
    sudo mkdir -p /etc/alertmanager
    sudo cp alertmanager-0.20.0.linux-amd64/alertmanager.yml /etc/alertmanager
    sudo chown -R alertmanager:alertmanager /etc/alertmanager
    sudo mkdir -p /var/lib/alertmanager
    sudo chown alertmanager:alertmanager /var/lib/alertmanager
    ```

### Add Configuration

Here's a basic example configuration. Customize it according to your requirements:

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: 'xxxxxx@outlook.com'
        from: 'xxxxxx@gmail.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'xxxxxxx@gmail.com'
        auth_password: 'xxxxxxxxxxxxxxxxx'
```

### Create a systemd Unit for Alertmanager

```bash
sudo vi /etc/systemd/system/alertmanager.service
```

Add the following content:

```ini
[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=alertmanager
Group=alertmanager
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager

[Install]
WantedBy=multi-user.target
```

### Start Alertmanager

```bash
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
sudo systemctl status alertmanager
curl localhost:9093
```

I could also access Alertmanager in a web browser at `http://localhost:9093`.

<img width="763" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/2397a800-4ce2-4adc-9d32-7108a6a46211">

### Install `amtool`

1. Install the amtool binary:

    ```bash
    sudo cp alertmanager-0.20.0.linux-amd64/amtool /usr/local/bin/
    ```

2. Create a config file for amtool:

    ```bash
    sudo mkdir -p /etc/amtool
    sudo vi /etc/amtool/config.yml
    ```

Enter the following content in the config file:

```yaml
alertmanager.url: http://localhost:9093 OR http://172.31.206.231:9093/#/alerts
```

3. Verify amtool is working by pulling the current Alertmanager configuration:

    ```bash
    amtool config show
    ```

## Configure Prometheus to Use Alertmanager

### Edit the Prometheus Config

```bash
sudo vi /etc/prometheus/prometheus.yml
```

Under `alerting`, add your Alertmanager as a target:

```yaml
# my global config
global:
  scrape_interval: 15s 
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
      - targets: ["localhost:9093"]  

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
   - "/etc/prometheus/alert.rules"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
  - job_name: "custom_metrics"
    static_configs:
      - targets: ["localhost:8000"]
```

### Restart Prometheus

```bash
sudo systemctl restart prometheus
```

### Verify Prometheus is Able to Reach the Alertmanager

Access the Prometheus Expression Browser in a web browser at `http://172.31.206.231:9090/graph`. 

<img width="568" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/1fefacbf-0fba-4a3b-a99a-188c55cd0c49">

Run the following query and ensure the current value is 1:

```prometheus
prometheus_notifications_alertmanagers_discovered =1
```
<img width="856" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a5523ef2-0125-41b6-92e9-91e298e6ab37">

## Setup SMTP for Gmail

### One Needs to Have 2-Step Verification Enabled for Google Account as Less Secure Apps has been deprecated for Google accounts.

1. Navigate to `https://myaccount.google.com/apppasswords`.
2. Name your app, then click the `Create` button.
3. A new app password will be generated for you. Copy this password.

## Configure SMTP on Grafana

### Open `/etc/grafana.ini` File

Paste this content and update the values with your own:

```ini
[smtp]
enabled = true
host = smtp.gmail.com:587
user = xxxxxxxx@gmail.com
password = xxxxxxxxxxxx
skip_verify = true
from_address = xxxxxxx@gmail.com
from_name = Grafana
```

### Once Updated, Restart Your Grafana Server

## Configure Contact Point in Grafana interface

### Create a List of Email IDs

This can be done with the help of Contact Point:

1. Select `Home` -> `Alerting` -> `Contact Points`.
2. Click `Add Contact Point`.
3. Enter the name of your Contact Point.
4. Select `Email/AlertManager` as the Integration.
5. Enter the list of email IDs as comma-separated values.
6. Test your SMTP Configuration and Contact Point by sending a test email by clicking `Test` in the right top corner.
7. If everything goes well, you’ll receive an email to the email IDs provided with the subject `[FIRING:1] (TestAlert Grafana)`.

<img width="582" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/649b9e31-3561-48e2-885d-2c6e00229a8d">

9. Click `Save Contact Point` to save.

**I set alert rule initially stated above for an alert to be triggered and email sent, once memory is above 10%, it worked, se screenshot below:**
<img width="643" alt="Screenshot 2024-06-10 233829" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a13997d6-eff4-40b7-a5b0-2aaf08820184">

## Configure Notification Policies

### Update the Default Policy

1. There will be a default Policy available in Grafana.
2. Click the 3 dots […] & choose `edit`.
3. Update the Contact Point to the one we have created.
4. Click on `Update default policy`.
<img width="841" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/98eb76f3-d5e2-4279-91bc-e680a25b2650">

## Create Grafana Dashboards

1. **Log in to Grafana** (default is `http://localhost:3000`).

2. **Add Prometheus as a Data Source**:
    - Go to `Configuration` > `Data Sources` > `Add data source`.
    - Select `Prometheus` and set the URL to `http://localhost:9090`.
    - Click on `Save & Test`.

### Add Dashboard in Grafana

1. Click on `Dashboard` → `+` symbol → `Import Dashboard`.
2. Use ID `1860` and click on `Load`. [This is the dashboard for Node Exporter in Grafana.com database].
3. Select `Prometheus` as the endpoint under Prometheus "data sources" dropdown.
4. Click `Import`.
5. This will show the monitoring dashboard [CPU, memory, disk] for the node.

<img width="841" alt="image" src="https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/6d9f97bc-6368-4d6b-8016-2961f1037f1e">


```
## Summary
By following these documentation, I have successfully set up a monitoring system that includes Prometheus for metrics collection, Grafana for visualization, and Alertmanager for notifications. I was able to monitor system performance, visualize key metrics, and receive alerts for critical conditions.

