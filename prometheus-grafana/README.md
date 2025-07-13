# Prometheus, Grafana & Alertmanager Monitoring Stack

A complete monitoring and alerting solution using Prometheus, Grafana, and Alertmanager with Docker Compose.

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Ports 9090, 3000, 9093, and 9091 available

### Quick Commands

```bash
sudo chown -R 472:472 data/grafana
sudo chown -R 65534:65534 data/prometheus

# Start the monitoring stack
docker-compose up -d

# Stop the monitoring stack
docker-compose down

# View logs
docker-compose logs

# Check status
docker-compose ps
```

### 1. Clone and Setup

```bash
git clone <your-repo-url>
cd prometheus-grafana
```

### 2. Start the Monitoring Stack

```bash
docker-compose up -d
```

### 3. Access the Services

- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)
- **Alertmanager**: http://localhost:9093
- **Prometheus Pushgateway**: http://localhost:9091

## 📁 Project Structure

```
prometheus-grafana/
├── docker-compose.yml
├── README.md
├── docs/
│   ├── prometheus.md
│   ├── grafana-docker-guide.md
│   ├── grafana-alerts.md
│   └── monitoring-gitlab-with-prometheus.md
├── config/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── alert_rules.yml
│   ├── alertmanager/
│   │   └── alertmanager.yml
│   └── grafana/
│       └── provisioning/
│           ├── dashboards/
│           │   └── dashboard.yml
│           └── datasources/
│               └── prometheus.yml
└── data/
    ├── prometheus/
    ├── grafana/
    └── alertmanager/
```

## 🐳 Docker Compose Configuration

### Main Docker Compose File

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./config/prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml
      - ./data/prometheus:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - ./data/grafana:/var/lib/grafana
      - ./config/grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./config/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - ./data/alertmanager:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    restart: unless-stopped

  pushgateway:
    image: prom/pushgateway:latest
    container_name: pushgateway
    ports:
      - "9091:9091"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
  alertmanager_data:
```

## ⚙️ Configuration Files

### Prometheus Configuration (`config/prometheus/prometheus.yml`)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['pushgateway:9091']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'gitlab'
    metrics_path: '/-/metrics'
    static_configs:
      - targets: ['your-gitlab-host:9090']
```

### Alert Rules (`config/prometheus/alert_rules.yml`)

```yaml
groups:
  - name: example
    rules:
      - alert: HighCpuUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% for more than 1 minute"

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 85% for more than 1 minute"
```

### Alertmanager Configuration (`config/alertmanager/alertmanager.yml`)

```yaml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alertmanager@example.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

## 📊 Grafana Setup

### Initial Login
- URL: http://localhost:3000
- Username: `admin`
- Password: `admin`

### Add Prometheus Data Source
1. Go to Configuration → Data Sources
2. Click "Add data source"
3. Select "Prometheus"
4. Set URL: `http://prometheus:9090`
5. Click "Save & Test"

### Import Dashboards
1. Go to Dashboards → Import
2. Import popular dashboards:
   - Node Exporter Full (ID: 1860)
   - Prometheus 2.0 Overview (ID: 3662)

## 🔧 Management Commands

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### View Logs
```bash
# All services
docker-compose logs

# Specific service
docker-compose logs prometheus
docker-compose logs grafana
docker-compose logs alertmanager
```

### Restart Services
```bash
docker-compose restart
```

### Update and Restart
```bash
docker-compose pull
docker-compose up -d
```

## 📈 Monitoring Targets

### Self-Monitoring
- Prometheus monitors itself
- Grafana dashboards show stack health

### External Targets
- Node Exporter (system metrics)
- GitLab (if configured)
- Custom applications via Pushgateway

## 🚨 Alerting

### Alert Rules
- CPU usage > 80% for 1 minute
- Memory usage > 85% for 1 minute
- Custom rules in `alert_rules.yml`

### Notification Channels
- Webhook (default)
- Email (configure SMTP)
- Slack (add configuration)

## 🔍 Troubleshooting

### Check Service Status
```bash
docker-compose ps
```

### Verify Prometheus Targets
1. Go to http://localhost:9090
2. Navigate to Status → Targets
3. Check target states

### Check Alertmanager
1. Go to http://localhost:9093
2. Verify alert configuration

### View Service Logs
```bash
# Prometheus logs
docker-compose logs prometheus

# Grafana logs
docker-compose logs grafana

# Alertmanager logs
docker-compose logs alertmanager
```

## 📚 Documentation

- [Prometheus Setup Guide](docs/prometheus.md)
- [Grafana Docker Guide](docs/grafana-docker-guide.md)
- [Grafana Alerts Configuration](docs/grafana-alerts.md)
- [GitLab Monitoring](docs/monitoring-gitlab-with-prometheus.md)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

This project is licensed under the MIT License.

---

**Happy Monitoring! 🎉** 