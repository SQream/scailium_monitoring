# scailium_monitoring
NOTE: Before you start, make sure that SElinux is disabled on all involved servers
NOTE: It is recommended to install Prometheus and Grafana servers separate machine - GPU not required.

Download from https://sq-ftp-public.s3.us-east-1.amazonaws.com/scailium_monitoring_20260215.tar.gz
tar -xf scailium_monitoring_20260215.tar.gz
File Structure:
scailium_monitoring
├── dashboards
│   ├── 8378_rev4_totalcpu_fix.json
│   ├── gpu_sanitised_dashboard.json
│   └── node-exporter-full_rev31.json
├── exporters
│   ├── Config
│   │   ├── alertmanager.yml
│   │   ├── alert_rules.yml
│   │   └── prometheus.yml
│   ├── node_exporter
│   │   └── node_exporter
│   ├── nvidia_exporter
│   │   └── nvidia_exporter
│   ├── process_exporter
│   │   ├── process-exporter_0.5.0_linux_amd64.rpm
│   │   └── process-exporter_0.6.0_linux_amd64.deb
│   ├── prometheus-exporters-install.sh
│   ├── README.md
│   └── services
│       ├── node_exporter.service
│       └── nvidia_exporter.service
├── grafana-enterprise_12.3.2+security-01_21834523998_linux_amd64.rpm
├── prometheus-server
│   ├── prometheus-2.31.1.linux-amd64.tar.gz
│   └── prometheus-server-install.sh
└── README.txt
USAGE:
Install Prometheus Server:
cd scailium_monitoring/prometheus-server/
./prometheus-server-install.sh     # Requires sudo permissions

The installer will:
    Create user prometheus
    Install Prometheus server
    Set correct permissions
    Create configuration file:
/etc/prometheus/prometheus.yml
Default content:
# node_exporter port : 9100
# nvidia_exporter port: 9445
# process-exporter port: 9256

global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets:
        - <prometheus server IP>:9090

  - job_name: 'processes'
    scrape_interval: 5s
    static_configs:
      - targets:
        - <process exporter IP>:9256
        - <another process exporter IP>:9256

  - job_name: 'nvidia'
    scrape_interval: 5s
    static_configs:
      - targets:
        - <nvidia exporter IP>:9445
        - <another nvidia exporter IP>:9445

  - job_name: 'nodes'
    scrape_interval: 5s
    static_configs:
      - targets:
        - <node exporter IP>:9100
        - <another node exporter IP>:9100

Edit Configuration:
sudo vi /etc/prometheus/prometheus.yml
Replace placeholders:
    Replace <prometheus server IP> with the real IPv4 address of your Prometheus server.
    Replace <process exporter IP> with the first server where exporters are installed.
    Replace <another ... IP> with the second server IP (if applicable).
If there is only one server to monitor, completely remove these lines:
    <another process exporter IP>:9256
    <another nvidia exporter IP>:9445
    <another node exporter IP>:9100
Restart Prometheus:
sudo systemctl daemon-reload
sudo systemctl restart prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus

Firewall (if enabled):
Open TCP port: 9090

Install Exporters on GPU servers:
cd cailium_monitoring/exporters
./prometheus-exporters-install.sh   # Requires sudo permissions
This installs:
    node_exporter
    nvidia_exporter
    process-exporter

Start and enable services:
sudo systemctl start node_exporter && sudo systemctl enable node_exporter
sudo systemctl start nvidia_exporter && sudo systemctl enable nvidia_exporter
sudo systemctl start process-exporter && sudo systemctl enable process-exporter

Verify services:
sudo systemctl status node_exporter
sudo systemctl status nvidia_exporter
sudo systemctl status process-exporter

Firewall (if enabled):
Open TCP ports:
  9100   (node_exporter)
  9445   (nvidia_exporter)
  9256   (process-exporter)

Install Grafana ( rpm file included in tarball ):
sudo dnf localinstall grafana-enterprise_12.3.2+security-01_21834523998_linux_amd64.rpm

Start Grafana:
sudo systemctl start grafana-server && sudo systemctl enable grafana-server

Connect to Grafana:
http://<grafana server IP>:3000

Default credentials: admin / admin
Add Prometheus Data Source:
    Go to Settings
    (or Connections in newer versions)
    Click Data Sources
    Click Add Data Source
    Choose Prometheus
    Set URL: http://<prometheus server IP>:9090
    Click Save & Test

Import Dashboards:
    Go to Dashboards
    Click New → Import
    Click Upload JSON file
    Navigate to the dashboards folder in exctracted tarball
    Import the JSON files one by one
Critically Important !!!
When importing each dashboard:
Under Data Source (or DS_PROMETHEUS in newer versions) dropdown select Prometheus
If you skip this step → dashboards will show No data

Firewall (if enabled):
Open TCP port: 3000

Final Ports Summary ( all TCP ):
Prometheus server -  9090
node_exporter     -  9100
nvidia_exporter   -  9445
process-exporter  -  9256

