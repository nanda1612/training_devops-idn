# Prometheus

## Install Prometheus on Docker
01. Create Directory and File configuration
```
mkdir /prometheus
touch /prometheus/prometheus.yml
```

02. Run Prometheus on Container
```
docker run -d --name prometheus-container -e TZ=UTC -p 9090:9090 --restart=always -v /prometheus/:/etc/prometheus/ -v prometheus-data:/prometheus prom/prometheus
```

03. Access prometheus ui
```
http://172.23.x.x:9090
```

04. Query prometheus
```
prometheus_target_interval_length_seconds
```

## Setup Exporter
### A. Node Exporter
Download node exporter package
```
docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  --name ct-node-exporter \
  --restart=always \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host 
```

Access node metric
```
http://ip-address:9100/metrics
```

Config prometheus
```
nano /prometheus/prometheus.yml
```

Add Targets
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "Server monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["172.23.x.x:9100", "172.23.x.x:9100", "172.23.x.x:9100"]
        labels:
          os: 'linux'
```

```
docker restart prometheus-container
```

Query metric
```
rate(node_cpu_seconds_total{mode="system"}[1m])
```

### B. Docker Cadvisor Exporter
Running cAdvisor in a Docker Container
```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:v0.49.1
```
cAdvisor is now running (in the background) on
```
http://172.23.x.x:8080/
```

Config prometheus
```
nano /prometheus/prometheus.yml
```

Add targets
```
- job_name: "Docker monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["172.23.x.x:8080"]
        labels:
          app: 'docker'
```

Restart Service
```
docker restart prometheus-container
```

Access prometheus ui
```
http://172.23.x.x:9090/
```

Query metric
```
rate(container_cpu_usage_seconds_total{name="redis"}[1m])
```
### C. Nginx Exporter
Edit nginx config
```
nano /etc/nginx/sites-available/default.conf
```

Add metrics 
```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }    
    // add this
    location /metrics {
        stub_status;
    }
}
```

Deploy nginx exporter
```
docker container run -d --rm -p 9113:9113 --name nginx-exporter nginx/nginx-prometheus-exporter -nginx.scrape-uri http://172.23.x.x/metrics
```

Config prometheus
```
nano /prometheus/prometheus.yml
```

Add targets
```
- job_name: "nginx monitoring"
    scrape_interval: 5s
    static_configs:
      - targets: ["172.23.x.x:9113"]
        labels:
          group: 'nginx'
```

Restart Service
```
docker restart prometheus-container
```

Access prometheus ui
```
http://172.23.x.x:9090/
```

Query metric
```
nginx_connections_active
```
## Setup rule
### A. Setup Record Rule
```
nano /prometheus/prometheus.rules.yml
```

```
groups:
  - name: cpu-node
    rules:
    - record: node_cpu
      expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

```
nano /prometheus/prometheus.yml
```

```
rule_files:
  - prometheus.rules.yml
```

```
docker restart prometheus-container
```

### B. Setup Alert Rule
Alert Server is down
```
nano /prometheus/prometheus.rules.yml
```
Add config
```
  - name: server_down
    rules:
    - alert: server_down
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: Server are down.
```

```
docker restart prometheus-container
```

### C. Alert CPU and memory > 50%
```
nano /prometheus/cpu.rules.yml
```

```
groups:
  - name: example_alerts
    rules:
    - alert: HighCPUUtilization
      expr: avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[1m]) * 100) < 50
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: High CPU utilization on host {{ $labels.instance  }}
        description: The CPU utilization on host {{ $labels.instance }} has exceeded 50 for 1 minutes.
```

```
nano /prometheus/prometheus.yml
```

```
rule_files:
  - prometheus.rules.yml
  - cpu.rules.yml
```

```
docker restart prometheus-container
```
