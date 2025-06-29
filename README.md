# LAN Monitoring
Configuration files for running a Prometheus server with snmp_exporter and node_exporter to collect metrics from a Synology NAS. The plan is for this to run on a Raspberry Pi but could run on any Linux system. Metrics will be remote-written to Mimir in Grafana Cloud.

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
wget https://github.com/prometheus/prometheus/releases/download/vX.Y.Z/prometheus-X.Y.Z.linux-amd64.tar.gz
tar -xzvf prometheus-X.Y.Z.linux-amd64.tar.gz
sudo mv prometheus-X.Y.Z.linux-amd64 /opt/prometheus
sudo chown -R prometheus:prometheus /opt/prometheus
```

- Symlink the config files
```bash
sudo ln -s ./prometheus/prometheus.yaml /etc/prometheus/prometheus.yaml
sudo ln -s ./prometheus/prometheus.service /etc/systemd/system/prometheus.service
```

### Node Exporter
- [Download](https://prometheus.io/download/##node_exporter) and install:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v<VERSION>/node_exporter-X-Y-Z.tar.gz
tar xvfz node_exporter-X-Y-Z.tar.gz
sudo mv node_exporter-X-Y-Z /opt/node_exporter
sudo chown -R prometheus:prometheus /opt/node_exporter
```

- Symlink the config files
```bash
sudo ln -s ./node_exporter/node_exporter.service /etc/systemd/system/node_exporter.service
```

### SNMP Exporter

#### Prerequisites

In order to use snmp_exporter, you need to have enabled SNMP on your Synology NAS. You also need to clone the snmp_exporter repository from GitHub and follow the instructions in the README file in the generator directory to generate the configuration file (see generator.BAK.yaml in snmp_exporter directory which was used to generate the snmp.yaml file in this repository). NB. These steps need to be followed on a linux system, it does not build on a macOS system. Running the generator will generate a snmp.yaml file that you can use as a starting point for your configuration. The snmp.yaml file in this repository was generate in this way.

#### Installation

- Copy env file and fill it with your SNMP auth values from the Synology NAS:
```bash
cp ./snmp_exporter/.env.example ./snmp_exporter/.env
vi ./snmp_exporter/.env

```
- Create directories:
```bash
sudo mkdir /etc/snmp_exporter
```



- [Download](https://github.com/prometheus/snmp_exporter/releases) and install:
```bash
wget https://github.com/prometheus/snmp_exporter/releases/download/v<VERSION>/snmp_exporter-X-Y-Z.tar.gz
tar xvfz snmp_exporter-X-Y-Z.tar.gz
sudo mv snmp_exporter-X-Y-Z /opt/snmp_exporter
sudo chown -R prometheus:prometheus /opt/snmp_exporter
```

- Symlink the snmp.yaml config file from this directory to /etc/snmp_exporter/snmp.yaml:
```bash
sudo ln -s ./snmp_exporter/snmp.yaml /etc/snmp_exporter/snmp.yaml
sudo ln -s ./snmp_exporter/.env /etc/snmp_exporter/.env
sudo ln -s ./snmp_exporter/snmp_exporter.service /etc/systemd/system/snmp_exporter.service
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

## References
- [Prometheus](https://prometheus.io/)
- [SNMP Exporter](https://github.com/prometheus/snmp_exporter)
- [Useful blog post by Colby.gg](https://colby.gg/posts/2023-10-17-monitoring-synology/)
