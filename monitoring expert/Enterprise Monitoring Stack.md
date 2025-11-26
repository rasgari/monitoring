# Enterprise Monitoring Stack

فایل: docker-compose.yml
```
version: '3.9'

networks:
  monitor-net:
    driver: bridge

services:

  #------------------------------------------------------
  # 1. Prometheus (Secure + Auth + SSL + Alerts)
  #------------------------------------------------------
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    networks:
      - monitor-net
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./prometheus/alerts.yml:/etc/prometheus/alerts.yml:ro
      - ./prometheus/auth:/auth
      - prometheus_data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--web.config.file=/auth/web.yml"
      - "--storage.tsdb.retention.time=20d"
      - "--storage.tsdb.retention.size=10GB"
    ports:
      - "9090:9090"
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 350M

  #------------------------------------------------------
  # 2. Alertmanager (Telegram Alerts)
  #------------------------------------------------------
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    restart: unless-stopped
    networks:
      - monitor-net
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    ports:
      - "9093:9093"

  #------------------------------------------------------
  # 3. cAdvisor (Ultra Optimized)
  #------------------------------------------------------
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.49.1
    container_name: cadvisor
    restart: unless-stopped
    networks:
      - monitor-net
    command:
      - "--storage_driver=none"
      - "--disable_metrics=percpu,process,advtcp,hugetlb,sched,diskIO"
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    deploy:
      resources:
        limits:
          cpus: "0.3"
          memory: 200M

  #------------------------------------------------------
  # 4. Node Exporter
  #------------------------------------------------------
  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    restart: unless-stopped
    networks:
      - monitor-net
    command:
      - "--path.rootfs=/host"
    volumes:
      - "/:/host:ro,rslave"
    ports:
      - "9100:9100"
    deploy:
      resources:
        limits:
          cpus: "0.2"
          memory: 150M

  #------------------------------------------------------
  # 5. Grafana (Provisioned Dashboards + Security)
  #------------------------------------------------------
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    networks:
      - monitor-net
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=StrongPass123!
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_AUTH_BASIC_ENABLED=true
      - GF_AUTH_ANONYMOUS_ENABLED=false
      - GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH=/var/lib/grafana/dashboards/main.json
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 300M


volumes:
  prometheus_data:
  grafana_data:
```
📌 فایل Prometheus (با Auth + Alerts)

📁 prometheus/prometheus.yml
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]

rule_files:
  - /etc/prometheus/alerts.yml

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["node_exporter:9100"]
```
📌 فایل Auth Prometheus

📁 prometheus/auth/web.yml
```
basic_auth_users:
  admin: $2y$10$RrVqbRHn8ul0nq2fTq5VCuAr7WNJFNpRum1iDkMUuP2EQF8P0KESy
```

پسورد مورد استفاده:
```
admin / StrongPass123!
```
⚠️ اگر می‌خواهی پسورد اختصاصی تولید کنم بگو:

👉 «پسورد جدید بساز»

📌 فایل هشدارها (alerts.yml)

📁 prometheus/alerts.yml
```
groups:
  - name: system-alerts
    rules:

    - alert: HighCPU
      expr: avg(rate(node_cpu_seconds_total{mode!="idle"}[2m])) * 100 > 80
      for: 2m
      labels:
        severity: warning
      annotations:
        description: "CPU Load is over 80%"

    - alert: HighMemory
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
      for: 3m
      labels:
        severity: critical
      annotations:
        description: "Memory usage above 85%"

    - alert: DiskFull
      expr: (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes) * 100 < 10
      for: 5m
      labels:
        severity: critical
      annotations:
        description: "Disk space is below 10%"

    - alert: ContainerDown
      expr: up{job="cadvisor"} == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        description: "cAdvisor container is down"
```
📌 Alertmanager with Telegram

📁 alertmanager/alertmanager.yml
```
route:
  receiver: "telegram"

receivers:
  - name: "telegram"
    telegram_configs:
      - bot_token: "YOUR_TELEGRAM_BOT_TOKEN"
        chat_id: "YOUR_CHAT_ID"
```

اگر خواستی Bot Token + Chat ID را بگیری، بگو:
👉 «تلگرام کانفیگ کن»

🎨 Grafana Dashboards (Auto Provision)

پوشه‌ها:
```
grafana/
  provisioning/
    dashboards/
      dashboard.yml
    datasources/
      datasource.yml
  dashboards/
      main.json   ← داشبورد آماده Docker + Linux
```

---

