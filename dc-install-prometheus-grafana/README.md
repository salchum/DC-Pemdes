# Guide to Install Prometheus and Grafana on Docker in Ubuntu 24.04
---
## **Step 1: Create Directory**
a. Create Directory for Data Vizualization
```
cd /srv/dockerdata
mkdir dataviz
```
b. Create Directory for ``Prometheus``
```
cd /srv/dockerdata/dataviz
mkdir prometheus
```
c. Create Directory for ``Grafana``
```
cd /srv/dockerdata/dataviz
mkdir grafana
```
d. Create Directory for ``Node Exporter``
```
cd /srv/dockerdata/dataviz
mkdir node-exporter
cd ./node-exporter
mkdir proc
mkdir sys
```

## **Step 2: Create prometheus.yml**
a. Create file ``prometheus.yml``
```
cd /srv/dockerdata/dataviz/prometheus
nano prometheus.yml
```
b. Copy and paste script below
```
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 1m
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

## **Step 3: Deploy using Docker Compose**
a. Create ``docker-compese.yml`` or use script below
```
networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}
  grafana_data: {}

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: monitoring_prometheus
    restart: unless-stopped
    ports:
      - 9090:9090
    volumes:
      - /srv/dockerdata/dataviz/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana
    container_name: monitoring_grafana
    restart: unless-stopped
    volumes:
      - grafana_data:/var/lib/grafana
    ports:
      - 3000:3000
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: monitoring_node-exporter
    restart: unless-stopped
    volumes:
      - /srv/dockerdata/dataviz/node-exporter/proc:/host/proc:ro
      - /srv/dockerdata/dataviz/node-exporter/sys:/host/sys:ro
      - /srv/dockerdata/dataviz/node-exporter/:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    expose:
      - 9100
    networks:
      - monitoring
```