# 📊 Phase 2 - Observability Stack (LGMA)

## 📝 Global Overview

The objective of this phase is to deploy a complete observability stack in a Docker environment in order to monitor:

- **System metrics**
- **Container metrics**
- **Application logs**
- **GeoIP analysis of web requests**

This architecture is based on six components:

- **Grafana** → Visualization
- **Prometheus** → Metrics collection
- **Loki** → Log centralization
- **Grafana Alloy** → Controlled log collection
- **Node Exporter** → System metrics
- **cAdvisor** → Docker metrics

> [!IMPORTANT]
> Full configuration files are not published for security reasons (production environment).
> Only the methodology and validation steps are provided.

---

# 🔧 Section 1: Docker Installation (Official Secure Method)

## 🔄 Update & Dependencies

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
```

## 🔐 Add Docker Keyring

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## 📦 Add Docker Repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo $UBUNTU_CODENAME) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 🐳 Install Docker Engine + Compose

```bash
sudo apt-get update
sudo apt-get install -y \
docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin
```

---

# 📂 Section 2: Observability Environment Preparation

Creation of a dedicated directory structure:

```bash
sudo mkdir -p /opt/observability/{grafana/provisioning/datasources,prometheus,loki,alloy,data}
```

## 📁 Configuration Files Created

The following files were configured:

```bash
/opt/observability/docker-compose.yml
/opt/observability/prometheus/prometheus.yml
/opt/observability/loki/loki.yml
/opt/observability/grafana/provisioning/datasources/datasources.yml
/opt/observability/alloy/config.alloy
```

These files ensure:

- Service orchestration
- Metrics scraping
- Persistent log storage
- Automatic datasource provisioning
- Controlled log collection

---

# 🐙 Section 3: Docker Compose Orchestration

Deployed services:

- grafana
- prometheus
- loki
- alloy
- node_exporter
- cadvisor

---

# 🚀 Section 4: Startup & Validation

## ▶️ Launch the Stack

```bash
cd /opt/observability
sudo docker compose up -d
```

## 🔍 Verify Containers

```bash
sudo docker compose ps
```

### 📸 Running Containers

![Docker Compose PS](../assets/monitoring_1.png)

All six containers are confirmed **Up** and operational.

---

# 📈 Section 5: System & Container Monitoring

## 🖥️ System Dashboard (Node Exporter – ID 1860)

Monitoring:

- CPU
- RAM
- Disk
- Network

![Node Exporter Dashboard](../assets/monitoring_2.png)

---

## 🐳 Docker Dashboard (cAdvisor)

Tracks CPU and memory usage per container.

![Docker cAdvisor Dashboard](../assets/monitoring_3.png)

> ℹ️ Most panels in these dashboards were manually reviewed, adapted, and optimized.
> They remain fully customizable to fit business or technical requirements.

---

# 📜 Section 6: NGINX Log Centralization (Loki)

## 🔎 Log Exploration in Grafana

Query built using labels:

```bash
{job="nginx", stream="access_json", service_name="nginx"}
```

![Loki Explore Interface](../assets/monitoring_5.png)

---

## 🌍 GeoIP-Enriched JSON Logs

Logs are enriched directly in NGINX with:

- geoip_country_name
- geoip_country_code
- HTTP method
- status code

![GeoIP Enriched JSON Logs](../assets/monitoring_6.png)

---

# 📊 Section 7: Example of GeoIP Panel Creation

The following demonstrates **a quick example of creating a custom panel** in Grafana.

## ➕ Add a New Panel

Menu Add → Visualization

![Add Panel](../assets/monitoring_7.png)

---

## 🧠 LogQL Query Used

```bash
sum by (geoip_country_code) (
  count_over_time(
    {job="nginx", stream="access_json"}
    | json
    | __error__=""
    [$__interval]
  )
)
```

## 📈 Result: GeoIP Dashboard

Bar Chart panel displaying log volume by country.

![GeoIP Dashboard Result](../assets/monitoring_12.png)

> ⚙️ As with the previous dashboards, panels are customizable and adjustable to improve readability and operational relevance.

---

# ✅ Section 8: Final Observability Chain Validation

Complete validation:

NGINX → Alloy → Loki → Grafana  
Node Exporter / cAdvisor → Prometheus → Grafana  

Final consolidated dashboard view:

![Final Consolidated View](../assets/monitoring_13.png)

---

# 🚀 Conclusion

The observability stack is now:

- Functional (metrics + logs)
- Correlated (logs + metrics + GeoIP)
- Secure (Grafana not publicly exposed)
- Persistent (Docker volumes)
- Reproducible (standardized YAML structure)

This infrastructure provides a solid foundation for future phases:

- Alerting
- Event correlation
- Anomaly detection
- Advanced Blue Team operations
