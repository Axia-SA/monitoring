# Prometheus + node-exporter + Grafana
Prometheus monitoring system with node-exporter and grafana docker containers

This stack contains three docker containers

| Container      | Function |
| ----------- | ----------- |
| `prometheus`      | Monitoring system and time series database       |
| `node-exporter`   | Prometheus exporter for hardware and OS metrics exposed by *NIX kernels, written in Go with pluggable metric collectors.        |
| `grafana`   | Operational dashboards for organize and show data        |

---

## docker-compose and configuration files

The `docker-compose.yml` file:
```yml
version: '3.8'

networks:
  monitoring:
    driver: bridge
    
volumes:
  prometheus_data: {}

services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - 9100:9100
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - /home/cristian/docker/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - /home/cristian/docker/prometheus/prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - monitoring
    
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    links:
      - prometheus:prometheus
    volumes:
      - /home/cristian/docker/prometheus/data/grafana:/var/lib/grafana
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=true
      - GF_SERVER_DOMAIN=mail.com
      - GF_SMTP_ENABLED=true
      - GF_SMTP_HOST=smtp.mail.com:port
      - GF_SMTP_USER=mail@mail.com
      - GF_SMTP_PASSWORD=password
      - GF_SMTP_FROM_ADDRESS=mail@mail.com

```

Before deploy the previous `docker-compose` file, check that [`prometheus.yml`](https://github.com/Axia-SA/monitoring/blob/main/prometheus.yml) configuration file exists on the `yml` path.


## exposed ports
 - Prometheus: `9090`
 - node-exporter: `9100`
 - Grafana: `3000`


## Deploy
After create `prometheus.yml` file deploy the stack with

```docker-compose up -d```

Then, access to [localhost:9090]() to check that Prometheus is active and running.

If all containers of this stack are running, acces to grafana at [localhost:3000]().


### Add the data source in grafana dashboard:

> In Configuration (cog icon) *(left panel)* -> Data sources -> add Prometheus

Form:

> Add URL `http://localhost:9090`

Next, import the dashboard in grafana. To this, we will use [Node Exporter Full](https://grafana.com/grafana/dashboards/1860)

> In Create *(plus icon)* -> Import -> add the ID `1860`

Done. The dashboard is ready to use.



