# monitoring export

---
کامل‌ترین و سبک‌ترین Docker Compose برای مانیتورینگ حرفه‌ای (Prometheus + cAdvisor + NodeExporter + Grafana) را می‌دهم که CPU و RAM حداقل ممکن باشد.


فایل تنظیمات Prometheus (ضروری)

📁 مسیر: ./prometheus/prometheus.yml
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]

  - job_name: "cadvisor"
    scrape_interval: 20s
    static_configs:
      - targets: ["cadvisor:8080"]

  - job_name: "node_exporter"
    scrape_interval: 20s
    static_configs:
      - targets: ["node_exporter:9100"]
```
🚀 نحوه اجرا
```
mkdir monitoring
cd monitoring
mkdir prometheus
nano prometheus/prometheus.yml
nano docker-compose.yml
docker compose up -d
```
🧪 مصرف منابع این Compose روی سرور 2 هسته / 4GB RAM
سرویس	CPU	RAM
```
cAdvisor	1–3%	100–180MB
Node Exporter	<1%	30–40MB
Prometheus	2–5%	200–300MB
Grafana	1–2%	100–200MB
```

---

