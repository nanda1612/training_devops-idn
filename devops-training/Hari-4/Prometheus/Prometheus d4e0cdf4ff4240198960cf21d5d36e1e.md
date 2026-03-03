# Prometheus

Created by: den
Created time: July 27, 2023 8:36 PM
Doc stage: New

# Install Prometheus

```bash
apt update
groupadd --system prometheus
useradd -s /sbin/nologin --system -g prometheus prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.45.5/prometheus-2.45.5.linux-amd64.tar.gz
tar vxf prometheus*.tar.gz
cd prometheus*/
mv prometheus /usr/local/bin
mv promtool /usr/local/bin
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
mv consoles /etc/prometheus
mv console_libraries /etc/prometheus
mv prometheus.yml /etc/prometheus
chown prometheus:prometheus /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
chown -R prometheus:prometheus /var/lib/prometheus

#### Edit target
vim /etc/prometheus/prometheus.yml

### create systemd
vim /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl enable --now prometheus
systemctl status prometheus
```

access prometheus ip-server:9090

![Screenshot 2023-07-27 at 21.16.26.png](Prometheus%20d4e0cdf4ff4240198960cf21d5d36e1e/Screenshot_2023-07-27_at_21.16.26.png)

ambil data/metrik

```bash
$ prometheus_target_interval_length_seconds
```

Untuk bisa memonitoring linux server, harus menginstall node_exporter.

prometheus jangan di stop, buat session ssh baru untuk konfigurasi node-exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar -zxvf node_exporter-1.7.0.linux-amd64.tar.gz
mv -v node_exporter-1.7.0.linux-amd64 /etc/node_exporter
```

```bash
sudo groupadd -f node_exporter
sudo useradd -g node_exporter --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /etc/node_exporter
```

vim /etc/systemd/system/node_exporter.service

```
[Unit]
Description=Node Exporter Service
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/etc/node_exporter/node_exporter --web.listen-address=:20000
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```jsx
systemctl daemon-reload
systemctl enable --now node_exporter
```

akses node exporter 

![Screenshot 2023-07-27 at 21.42.15.png](Prometheus%20d4e0cdf4ff4240198960cf21d5d36e1e/Screenshot_2023-07-27_at_21.42.15.png)

supaya prometheus bisa ambil metrix yang ada didalam node exporter, harus dikonfigurasi di .yml (ctrl+x) → proses prometheus, biarkan node expoeter berjalan

```bash
$ cd /etc/prometheus
$ vim prometheus
- job_name: "Server monitoring"
  scrape_interval: 5s
  static_configs:
    - targets: ["10.23.0.11:10000", "10.23.0.12:8001", "10.23.0.13:8003"]
      labels:
      group: 'devops'
$ ./prometheus --config.file=prometheus.yml
```

akses ke dashboard prometheus → status → Targets

![Screenshot 2023-07-27 at 21.59.10.png](Prometheus%20d4e0cdf4ff4240198960cf21d5d36e1e/Screenshot_2023-07-27_at_21.59.10.png)

check cpu graph

```bash
$ rate(node_cpu_seconds_total{mode="system"}[1m])
```

add node exporter di devops-1 & devops jenkins

monitoring docker

tambahkan di file prometheus.yml

Running cAdvisor in a Docker Container

```bash
docker run   --volume=/:/rootfs:ro   --volume=/var/run:/var/run:ro   --volume=/sys:/sys:ro   --volume=/var/lib/docker/:/var/lib/docker:ro   --volume=/dev/disk/:/dev/disk:ro   --publish=8080:8080   --detach=true   --name=cadvisor   --privileged   --device=/dev/kmsg   gcr.io/cadvisor/cadvisor:v0.47.2

```

akses dashboard ip:8008

edit file prometheus.yml

```bash
- job_name: "docker monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["10.23.0.113:8008"]
        labels:
          group: 'docker'
```

didashboard promethius

```bash
$ rate(container_cpu_usage_seconds_total{name="simple-a pps-apps-1"}[1m])
```

#