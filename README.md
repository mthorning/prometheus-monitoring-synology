# Monitoring Synology NAS with Prometheus (and Grafana)
Configuration files for running a Prometheus server with snmp_exporter and node_exporter to collect metrics from a Synology NAS. The plan is for this to run on a Raspberry Pi but could run on any Linux system. Metrics will be remote-written to Mimir in Grafana Cloud.

A note about network-monitor: This is a small tool I created to collect additional network metrics, but its use is optional. If you choose not to run network-monitor, simply update your prometheus.yaml configuration file to remove the scrape target for it. The rest of the setup will work as described without this component.

## Installation

- Create a dedicated user:
```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

### Prometheus

- Create directories:
```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

- Download & install the binary from [the prometheus.io download page](https://prometheus.io/download/):
```bash
P_VERSION=3.4.2 # Replace with desired version
P_OS_ARCH=linux-arm64 # Replace with desired OS-Arch
wget https://github.com/prometheus/prometheus/releases/download/v$P_VERSION/prometheus-$P_VERSION.$P_OS_ARCH.tar.gz
tar -xzvf prometheus-$P_VERSION.$P_OS_ARCH.tar.gz
sudo mv prometheus-$P_VERSION.$P_OS_ARCH/prometheus /opt/prometheus
sudo chown -R prometheus:prometheus /opt/prometheus
```

- Copy the config files
```bash
sudo cp ./prometheus/prometheus.yaml /etc/prometheus/prometheus.yaml
sudo cp ./prometheus/prometheus.service /etc/systemd/system/prometheus.service
```

- Update the prometheus.yaml file remote_write URL
```bash
sudo vi /etc/prometheus/prometheus.yaml
```

### Node Exporter
- [Download](https://prometheus.io/download/##node_exporter) and install:
```bash
N_VERSION=1.9.1 # Replace with desired version
N_OS_ARCH=linux-arm64 # Replace with desired OS-Arch
wget https://github.com/prometheus/node_exporter/releases/download/v$N_VERSION/node_exporter-$N_VERSION.$N_OS_ARCH.tar.gz
tar xvfz node_exporter-$N_VERSION.$N_OS_ARCH.tar.gz
sudo mv node_exporter-$N_VERSION.$N_OS_ARCH/node_exporter /opt/node_exporter
sudo chown -R prometheus:prometheus /opt/node_exporter
```

- Copy the config files
```bash
sudo cp ./node_exporter/node_exporter.service /etc/systemd/system/node_exporter.service
```

### SNMP Exporter

#### Prerequisites

In order to use snmp_exporter, you need to have enabled SNMP on your Synology NAS. You also need to clone the snmp_exporter repository from GitHub and follow the instructions in the README file in the generator directory to generate the configuration file (see generator.BAK.yaml in snmp_exporter directory which was used to generate the snmp.yaml file in this repository). NB. These steps need to be followed on a linux system, it does not build on a macOS system. Running the generator will generate a snmp.yaml file that you can use as a starting point for your configuration. The snmp.yaml file in this repository was generate in this way.

#### Installation

- Create directories:
```bash
sudo mkdir /etc/snmp_exporter
```

- [Download](https://github.com/prometheus/snmp_exporter/releases) and install:
```bash
S_VERSION=0.29.0 # Replace with desired version
S_OS_ARCH=linux-arm64 # Replace with desired OS-Arch
wget https://github.com/prometheus/snmp_exporter/releases/download/v$S_VERSION/snmp_exporter-$S_VERSION.$S_OS_ARCH.tar.gz
tar xvfz snmp_exporter-$S_VERSION.$S_OS_ARCH.tar.gz
sudo mv snmp_exporter-$S_VERSION.$S_OS_ARCH/snmp_exporter /opt/snmp_exporter
sudo chown -R prometheus:prometheus /opt/snmp_exporter
```

- Copy the config files
```bash
sudo cp ./snmp_exporter/snmp.yaml /etc/snmp_exporter/snmp.yaml
sudo cp ./snmp_exporter/snmp_exporter.service /etc/systemd/system/snmp_exporter.service
```

- Update the snmp.yaml file auth section with your SNMP auth values from the Synology NAS:
```bash
sudo vi /etc/snmp_exporter/snmp.yaml
```

## Start the services
- Reload systemd and start the services:
```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus node_exporter snmp_exporter
```

- Enable services to start on boot:
```bash
sudo systemctl enable prometheus node_exporter snmp_exporter
```

- Check the status of a service:
```bash
sudo systemctl status <SERVICE_NAME>
```
You should see "active (running)".

- View logs for a service (for troubleshooting):
```bash
sudo journalctl -u <SERVICE_NAME>.service -f
```

## Links
- [Prometheus](https://prometheus.io/)
- [SNMP Exporter](https://github.com/prometheus/snmp_exporter)
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [Useful blog post by Colby.gg](https://colby.gg/posts/2023-10-17-monitoring-synology/)
- [Synology dashboard for import](https://grafana.com/grafana/dashboards/13516-synology-snmp-dashboard/)
