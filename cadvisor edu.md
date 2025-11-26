# cadvisor edu

---
مشکل بالا بودن CPU در cadvisor


✅ علت‌های اصلی CPU بالا در cAdvisor

۱. Scrape زیاد Prometheus → هر ۵ ثانیه یا ۱ ثانیه
۲. جمع‌آوری متریک‌های سنگین (per-cpu, per-process, disk stats)
۳. کانتینرهای زیاد / پر I/O
۴. storage داخلی cAdvisor
۵. نسخه‌های دارای Memory/CPU leak

🎯 راه‌حل‌های ۱۰۰٪ کاربردی برای کم کردن CPU + RAM cAdvisor
⭐ ۱. غیرفعال کردن Storage داخلی cAdvisor (بیشترین تاثیر)

این مهم‌ترین کار است؛ روی CPU و RAM هر دو اثر دارد:
```
command:
  - "--storage_driver=none"
```

این بخش باعث می‌شود cAdvisor هیچ تاریخی ذخیره نکند → CPU تا ۴۰٪ کمتر.

⭐ ۲. کم کردن Scrape Prometheus (مشکل اصلی CPU بالا)

فایل prometheus.yml را باز کن:

اگر الان این است:
```
scrape_interval: 5s
```

تبدیل کن به:
```
scrape_interval: 15s
scrape_timeout: 10s
```

یا اگر سرور سنگین است:
```
scrape_interval: 30s
```

⚡ این تغییر مصرف CPU را ۵۰٪ کاهش می‌دهد.

⭐ ۳. غیرفعال کردن متریک‌های سنگین cAdvisor

این‌ها باعث سنگین شدن CPU می‌شوند:

percpu

process

advtcp

network

disk

hugetlb

نسخه سبک:
```
command:
  - "--storage_driver=none"
  - "--disable_metrics=percpu,process,advtcp,hugetlb,sched,diskIO"
```

ساده‌ترین روش کاهش مصرف CPU همین گزینه است.

⭐ ۴. محدود کردن CPU و RAM برای خود cAdvisor

حتماً محدودیت بگذار:

اجرای مستقیم:
```
docker run -d \
  --cpus=0.3 \
  --memory=200m \
  --memory-swap=200m \
  gcr.io/cadvisor/cadvisor:latest
```
در Docker Compose:
```
deploy:
  resources:
    limits:
      cpus: "0.3"
      memory: 200M
```
⭐ ۵. prune کانتینرهای قدیمی

اگر کانتینرهای dead زیاد دارید CPU مصرفی cAdvisor بالا می‌رود.

بررسی:
```
docker ps -a
```

حذف:
```
docker container prune
```
⭐ ۶. استفاده از نسخه پایدار بدون CPU leak

حتماً از نسخه زیر استفاده کن:
```
gcr.io/cadvisor/cadvisor:v0.49.1
```

نسخه‌های قبل CPU leak داشتند.

⭐ ۷. mountهای read-only استفاده کن (کاهش I/O → کاهش CPU)
```
volumes:
  - /:/rootfs:ro
  - /var/run:/var/run:ro
  - /sys:/sys:ro
  - /var/lib/docker/:/var/lib/docker:ro
```
🔥 نسخه نهایی بسیار سبک cAdvisor (CPU و RAM کم)

اگر خواستی همین را دقیق کپی کن:
```
cadvisor:
  image: gcr.io/cadvisor/cadvisor:v0.49.1
  container_name: cadvisor
  restart: unless-stopped
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
```
---
