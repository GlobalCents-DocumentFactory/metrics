# PTFD Observability Stack (DocumentFactory)

This documentation describes the observability stack for monitoring the `PTFDService.exe` Windows service.

## Core Components

The stack runs in Docker and monitors the service running natively on the host:

1.  **Prometheus**: Scrapes metrics from the `PTFDService.exe` endpoint.
2.  **Grafana**: Visualizes the metrics collected by Prometheus.
3.  **Jaeger**: Collects and visualizes distributed traces from the service.

## Quick Setup

1.  **Navigate to the docker folder**:
    ```bash
    cd docker
    ```

2.  **Start the stack**:
    ```bash
    docker-compose up -d
    ```

3.  **Access Interfaces**:
    - **Grafana**: [http://localhost:3000](http://localhost:3000) (Admin: `admin` / `ptfd123`)
    - **Prometheus**: [http://localhost:9090](http://localhost:9090)
    - **Jaeger**: [http://localhost:16686](http://localhost:16686)

## Metrics Collection

The `PTFDService.exe` exposes a Prometheus metrics endpoint on port **5123**. 
Prometheus is configured to scrape this via: `http://host.docker.internal:5123/metrics`

## Metrics Reference

Users can utilize the following metrics to create custom dashboards in Grafana.

### 1. Service Information
| Metric | Type | Labels | Description |
| :--- | :--- | :--- | :--- |
| `ptfdservice_info` | Gauge | `app_name`, `app_version`, `dotnet_version` | Static info about the running service instance. Value is always 1. |

### 2. Workflow Metrics
These metrics track high-level pipeline execution.
| Metric | Type | Labels | Description |
| :--- | :--- | :--- | :--- |
| `ptfd_workflow_counter` | Counter | `workflow_type` | Total number of workflows (pipelines) that have started processing. |
| `ptfd_workflow_errors` | Counter | `workflow_type` | Total number of workflows that resulted in an error. |
| `ptfd_workflow_latency` | Summary | `workflow_type` | Latency (in seconds) for complete workflow processing. |

### 3. Job Metrics
Jobs correspond to specific tasks within a workflow (e.g., Merge, OCR).
| Metric | Type | Labels | Description |
| :--- | :--- | :--- | :--- |
| `ptfd_job_counter` | Counter | `job_type`, `job_processortype` | Total number of jobs executed. |
| `ptfd_job_errors` | Counter | `job_type`, `job_processortype` | Total number of job-level failures. |
| `ptfd_job_latency` | Summary | `job_type`, `job_processortype` | Latency of individual job processing. |

### 4. Document Processor Metrics
Metrics for specific document operations like conversion or metadata collection.
| Metric | Type | Labels | Description |
| :--- | :--- | :--- | :--- |
| `ptfd_docprocessor_counter` | Counter | `processor_type`, `processor_action` | Number of document processor actions (e.g., pdf/convert). |
| `ptfd_docprocessor_errors` | Counter | `processor_type`, `processor_action` | Failures during document processing. |
| `ptfd_docprocessor_latency` | Summary | `processor_type`, `processor_action` | Time taken for document actions. |

### 5. Artifact Metrics
Tracking the movement of files to/from storage (usually NATS).
| Metric | Type | Labels | Description |
| :--- | :--- | :--- | :--- |
| `ptfd_artifact_counter` | Counter | `artifact_type`, `artifact_action` | Number of artifact operations (download/upload). |
| `ptfd_artifact_errors` | Counter | `artifact_type`, `artifact_action` | Errors during artifact persistence. |
| `ptfd_artifact_latency` | Summary | `artifact_type`, `artifact_action` | Time taken for artifact transport. |

### 6. System & Process Metrics
Standard resource tracking metrics provided by the runtime.
| Metric | Type | Description |
| :--- | :--- | :--- |
| `process_cpu_seconds_total` | Counter | Total user and system CPU time spent in seconds. |
| `process_resident_memory_bytes` | Gauge | Resident memory size (RSS) in bytes. |
| `process_virtual_memory_bytes` | Gauge | Virtual memory size in bytes. |
| `process_open_fds` | Gauge | Number of open file descriptors. |
| `dotnet_gc_collections_count_total` | Counter | Total GC collections per generation. |

## Error Investigation Workflow

When a pipeline fails, the user receives a **correlationId** (e.g., `47872a4f-2e22-4055-959d-ef5eec2aa518`). Admins can use this to trace through the system and identify the root cause.

### Step 1: Open the Error Analysis Dashboard
Navigate to: [PTFD Error Analysis](http://localhost:3000/d/ptfd-error-analysis/ptfd-error-analysis)

### Step 2: Enter the Correlation ID
Paste the user-provided `correlationId` into the input box at the top of the dashboard.

### Step 3: Find the Trace in Jaeger
The dashboard will attempt to search Jaeger for traces with the matching `messaging.conversation_id` tag.

Alternatively, open [Jaeger UI](http://localhost:16686/search) directly and search for:
- **Service**: `PTFD`
- **Tags**: `messaging.conversation_id=YOUR_CORRELATION_ID`

### Step 4: Analyze the Trace
- Look for spans with **error=true** status (highlighted in red).
- Check the `error.type` and `error.message` tags to understand the failure.
- Review the trace timeline to see which step in the pipeline failed.

### Step 5: Review Error Metrics
Use the Error Overview and Breakdown panels on the dashboard to understand:
- Which **component** is failing most often (Workflow, Job, Processor, Artifact).
- Which **document types** or **actions** are problematic.

## Fact Sheet

| Component | Port | Description |
| :--- | :--- | :--- |
| Prometheus | 9090 | Metrics Storage |
| Grafana | 3000 | Visualization |
| Jaeger | 16686 | Tracing UI |
| PTFD Metrics | 5123 | Exposed by `PTFDService.exe` |
