# 📊 PTFD Observability Stack

A complete observability stack for monitoring the **PTFDService.exe** (DocumentFactory) Windows service. This solution provides real-time metrics, distributed tracing, and log aggregation through a containerized monitoring infrastructure.

---

## 🏗️ Architecture

```javascript
┌─────────────────────────────────────────────────────────────────────────┐
│                           HOST MACHINE                                  │
│  ┌─────────────────┐                                                    │
│  │ PTFDService.exe │ ───▶ :5123/metrics                                │
│  └─────────────────┘                                                    │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────────────┐
│                         DOCKER NETWORK                                  │
│                                                                         │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                 │
│  │  Prometheus  │   │    Jaeger    │   │     Loki     │                 │
│  │    :9090     │   │   :16686     │   │    :3100     │                 │
│  │   (Metrics)  │   │  (Tracing)   │   │    (Logs)    │                 │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘                 │
│         │                  │                  │                         │
│         └──────────────────┼──────────────────┘                         │
│                            ▼                                            │
│                    ┌──────────────┐                                     │
│                    │   Grafana    │                                     │
│                    │    :3000     │                                     │
│                    │ (Dashboard)  │                                     │
│                    └──────────────┘                                     │
│                                                                         │
│  ┌──────────────┐                                                       │
│  │   Promtail   │ ◀── E:/docfactorydata/logs                            │
│  │ (Log Shipper)│                                                        │
│  └──────────────┘                                                        │
└──────────────────────────────────────────────────────────────────────────┘
```

## ✨ Features

- **📈 Metrics Collection** – Prometheus scrapes custom application metrics from PTFDService
- **🔍 Distributed Tracing** – Jaeger with OTLP support for end-to-end request tracing
- **📝 Log Aggregation** – Loki + Promtail for centralized log management
- **📊 Visualization** – Pre-configured Grafana dashboards for insights
- **🚨 Alerting** – Prometheus alerting rules for proactive monitoring

---

## 🚀 Quick Start

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- PTFDService.exe running on the host machine

### Start the Stack

```powershell
# Clone the repository
git clone https://github.com/GlobalCents-DocumentFactory/metrics.git
cd metrics

# Start all services
docker-compose up -d
```

### Access the Interfaces

| Service    | URL                                      | Credentials        |
|------------|------------------------------------------|--------------------|
| Grafana    | [localhost:3000](http://localhost:3000)  | `admin` / `ptfd123`|
| Prometheus | [localhost:9090](http://localhost:9090)  | –                  |
| Jaeger     | [localhost:16686](http://localhost:16686)| –                  |

---

## 📋 Available Metrics

PTFDService exposes metrics on port **5123** (`http://host.docker.internal:5123/metrics`).

### Service Information
| Metric | Description |
|--------|-------------|
| `ptfdservice_info` | Static service info (version, runtime) |

### Workflow Metrics
| Metric | Description |
|--------|-------------|
| `ptfd_workflow_counter` | Total workflows started |
| `ptfd_workflow_errors` | Workflow failures |
| `ptfd_workflow_latency` | Workflow processing time |

### Job Metrics
| Metric | Description |
|--------|-------------|
| `ptfd_job_counter` | Jobs executed |
| `ptfd_job_errors` | Job failures |
| `ptfd_job_latency` | Job processing time |

### Document Processor Metrics
| Metric | Description |
|--------|-------------|
| `ptfd_docprocessor_counter` | Document operations |
| `ptfd_docprocessor_errors` | Processing failures |
| `ptfd_docprocessor_latency` | Processing time |

### Artifact Metrics
| Metric | Description |
|--------|-------------|
| `ptfd_artifact_counter` | Artifact operations (up/download) |
| `ptfd_artifact_errors` | Artifact failures |
| `ptfd_artifact_latency` | Transfer time |

---

## 🔎 Error Investigation

When a pipeline fails, users receive a **correlationId**. Use this to trace through the system:

### 1️⃣ Open Error Analysis Dashboard
Navigate to: [PTFD Error Analysis](http://localhost:3000/d/ptfd-error-analysis/ptfd-error-analysis)

### 2️⃣ Enter Correlation ID
Paste the `correlationId` into the dashboard filter.

### 3️⃣ Find Trace in Jaeger
Search in [Jaeger UI](http://localhost:16686/search):
- **Service**: `PTFD`
- **Tags**: `messaging.conversation_id=YOUR_CORRELATION_ID`

### 4️⃣ Analyze the Trace
- Look for spans with **`error=true`** (highlighted in red)
- Check `error.type` and `error.message` tags
- Review the timeline to identify the failing step

---

## 📁 Project Structure

```
metrics/
├── config/
│   ├── grafana/
│   │   ├── dashboards/          # Pre-built Grafana dashboards
│   │   └── provisioning/        # Auto-provisioning configuration
│   ├── jaeger/
│   │   └── sampling_strategies.json
│   ├── prometheus/
│   │   ├── prometheus.yml       # Scrape configuration
│   │   └── rules/               # Alerting rules
│   └── promtail/
│       └── promtail.yml         # Log collection config
├── data/                        # Runtime data (gitignored)
├── logs/                        # Application logs (gitignored)
├── docs/
│   └── PTFD-Observability.md    # Detailed documentation
├── docker-compose.yml           # Stack orchestration
└── README.md
```

---

## ⚙️ Configuration

### Prometheus Targets

Edit `config/prometheus/prometheus.yml` to modify scrape targets:

```yaml
scrape_configs:
  - job_name: 'ptfd-service'
    static_configs:
      - targets: ['host.docker.internal:5123']
```

### Log Collection

Promtail is configured to collect logs from `E:/docfactorydata/logs`. Update `config/promtail/promtail.yml` to change the log path.

### Data Retention

| Component  | Default Retention |
|------------|-------------------|
| Prometheus | 30 days / 10GB    |
| Jaeger     | Persistent (Badger DB) |

---

## 🛑 Stop the Stack

```powershell
docker-compose down
```

To remove all data volumes:

```powershell
docker-compose down -v
```

---

## 📚 Documentation

For detailed documentation, see [docs/PTFD-Observability.md](docs/PTFD-Observability.md).

---

## 📄 License

© GlobalCents DocumentFactory
