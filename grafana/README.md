# Grafana Docker Setup

1. Create a persistent volume for your data:
   `docker volume create grafana-storage`
2. Start Grafana

```
docker run -d -p 3000:3000 --name=grafana \
  --volume grafana-storage:/var/lib/grafana \
  grafana/grafana-enterprise
```

# Prometheus & Node Exporter Docker Setup

1. Create a prometheus.yml and paste contents. (If you install node_exporter on more virtual machines, you need to update the yaml file with the additional targets.)

```
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

2. Create persistent volume
   `docker volume create prometheus_data`

3. Create Docker Compose file

```
version: '3.8'

volumes:
  prometheus_data: {}

services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - /tmp/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    network_mode: host
```

Issues Encountered
My Prometheus & Node Exporter container were running on different networks and when going to Prometheus, I was getting “Error scraping target: Get "http://localhost:9100/metrics": dial tcp [::1]:9100: connect: connection refused”. Prometheus network was set to grafana while node exporter was set to host. I ended up setting Prometheus to network_mode: host as well in the docker compose file.
