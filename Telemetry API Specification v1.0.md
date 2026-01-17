# Carbon Mirror Standard
## Telemetry API Specification v1.0

### I. Purpose and Scope

This document defines the technical requirements for companies seeking Obsidian Seal certification under the Carbon Mirror Standard. It specifies the exact telemetry data that must be exposed, the format in which it must be delivered, and the integration mechanisms for major cloud providers and on-premise infrastructure.

**Core Principle:** Obsidian certification requires **continuous, unforgeable telemetry** that the CMS Dashboard can verify independently. Companies cannot self-report metrics—data must be pulled directly from infrastructure monitoring systems.

---

### II. Telemetry Data Requirements

#### A. Ghost Compute Metrics

**Purpose:** Identify idling infrastructure consuming resources with minimal utilization.

**Required Data Points:**

```json
{
  "ghost_compute": {
    "instance_inventory": [
      {
        "instance_id": "i-0abc123def456",
        "instance_type": "m5.2xlarge",
        "region": "us-east-1",
        "state": "running",
        "launch_time": "2024-11-15T08:30:00Z",
        "cpu_utilization_30day_avg": 3.2,
        "memory_utilization_30day_avg": 8.1,
        "network_io_30day_avg_mbps": 0.4,
        "disk_io_30day_avg_iops": 12,
        "classification": "ghost",
        "monthly_cost_usd": 280.32
      }
    ],
    "summary": {
      "total_instances": 847,
      "ghost_instances": 34,
      "ghost_ratio": 0.040,
      "monthly_waste_usd": 9530.88
    }
  }
}
```

**Calculation Rules:**
- Instance classified as "ghost" if `cpu_utilization_30day_avg < 10%` AND `memory_utilization_30day_avg < 10%`
- Metrics calculated as rolling 30-day average, updated every 15 minutes
- All running instances must be reported (no cherry-picking)

**Data Source Integration:**
- **AWS:** CloudWatch Metrics API (`GetMetricStatistics`)
- **Azure:** Azure Monitor Metrics API
- **GCP:** Cloud Monitoring API (`timeSeries.list`)
- **On-Prem:** Prometheus, Grafana, or equivalent with `/metrics` endpoint

---

#### B. Model Efficiency Metrics

**Purpose:** Track whether routine tasks are being routed to appropriately-sized AI models.

**Required Data Points:**

```json
{
  "model_efficiency": {
    "inference_log": [
      {
        "request_id": "req_7f3a9b2c",
        "timestamp": "2025-01-16T14:23:11Z",
        "task_type": "classification",
        "task_complexity": "routine",
        "model_used": "gpt-4o",
        "model_parameters": 175000000000,
        "response_time_ms": 1240,
        "tokens_processed": 450,
        "energy_estimate_wh": 2.1
      },
      {
        "request_id": "req_8g4b0c3d",
        "timestamp": "2025-01-16T14:23:15Z",
        "task_type": "summarization",
        "task_complexity": "routine",
        "model_used": "llama-3-8b",
        "model_parameters": 8000000000,
        "response_time_ms": 380,
        "tokens_processed": 520,
        "energy_estimate_wh": 0.3
      }
    ],
    "summary": {
      "total_inference_requests_30day": 284720,
      "routine_tasks_total": 198904,
      "routine_to_slm_count": 142033,
      "routine_to_slm_ratio": 0.714,
      "model_mismatch_count": 56871,
      "energy_wasted_kwh": 1247.3
    }
  }
}
```

**Classification Rules:**

**Routine Tasks** (should use SLM ≤ 30B parameters):
- Text classification
- Sentiment analysis
- Named entity recognition
- Summarization (single document, <5000 tokens)
- Data extraction from structured formats
- Simple Q&A over knowledge bases
- Translation (common language pairs)

**Frontier Tasks** (may justify large models):
- Multi-step reasoning requiring chain-of-thought
- Code generation for complex systems
- Research synthesis across multiple sources
- Creative writing requiring nuanced style
- Multi-turn conversations requiring deep context
- Ambiguous problem-solving without clear solution paths

**Task Complexity Determination:**
- Companies must implement task classifier that tags each request as "routine" or "frontier"
- CMS Dashboard performs **spot-checking** by random sampling 1% of requests and re-classifying
- If spot-check shows >15% misclassification rate, company receives WARNING
- If misclassification persists for 14 days, Obsidian certification is revoked

**Data Source Integration:**
- Model inference logs from:
  - OpenAI API usage logs
  - Anthropic API usage logs
  - Local model serving platforms (vLLM, TGI, Ollama)
  - Custom inference endpoints (must expose standardized log format)

---

#### C. Dark Data Metrics

**Purpose:** Quantify storage waste from unutilized data.

**Required Data Points:**

```json
{
  "dark_data": {
    "storage_inventory": [
      {
        "bucket_name": "company-archive-2019",
        "region": "us-west-2",
        "total_size_gb": 4280.5,
        "object_count": 187340,
        "last_accessed_oldest": "2022-03-14T00:00:00Z",
        "days_since_access": 1038,
        "classification": "dark",
        "monthly_cost_usd": 98.25,
        "storage_class": "standard"
      },
      {
        "bucket_name": "company-logs-2024",
        "region": "us-west-2",
        "total_size_gb": 892.1,
        "object_count": 45023,
        "last_accessed_oldest": "2025-01-10T00:00:00Z",
        "days_since_access": 6,
        "classification": "active",
        "monthly_cost_usd": 20.47,
        "storage_class": "standard"
      }
    ],
    "summary": {
      "total_storage_gb": 52847.3,
      "dark_data_gb": 7914.2,
      "dark_data_ratio": 0.150,
      "monthly_waste_usd": 1815.73,
      "recommended_action": "migrate 7.9TB to cold storage"
    }
  }
}
```

**Calculation Rules:**
- Data classified as "dark" if `days_since_last_access > 180`
- "Last accessed" determined by object metadata (read operations, not just listing)
- Deleted data must be removed from calculations within 24 hours

**Anti-Gaming Protections:**
- CMS Dashboard tracks **access pattern anomalies**
- If >20% of dark data suddenly shows recent access without corresponding deletion or migration logs, flag for review
- Companies must provide deletion/migration audit trails on request

**Data Source Integration:**
- **AWS S3:** S3 Inventory + CloudTrail data events
- **Azure Blob Storage:** Storage Analytics logs
- **GCP Cloud Storage:** Storage Insights + Audit Logs
- **On-Prem:** File system metadata scanners (mtime tracking)

---

#### D. Heat Recovery Metrics

**Purpose:** Measure waste heat capture from computational infrastructure.

**Required Data Points (if thermal load ≥ 50kW):**

```json
{
  "heat_recovery": {
    "thermal_sources": [
      {
        "location": "datacenter-rack-07",
        "equipment_type": "gpu_cluster",
        "thermal_output_kw": 145.3,
        "exhaust_temp_celsius": 42.0,
        "cooling_method": "liquid_to_air_heat_exchanger",
        "heat_captured_kw": 38.2,
        "heat_utilization": "office_space_heating",
        "recovery_efficiency": 0.263
      },
      {
        "location": "datacenter-rack-12",
        "equipment_type": "server_rack",
        "thermal_output_kw": 68.7,
        "exhaust_temp_celsius": 38.0,
        "cooling_method": "air_conditioning",
        "heat_captured_kw": 0.0,
        "heat_utilization": "none",
        "recovery_efficiency": 0.0
      }
    ],
    "summary": {
      "total_thermal_output_kw": 892.4,
      "total_heat_captured_kw": 241.6,
      "overall_recovery_ratio": 0.271,
      "primary_utilization_methods": [
        "office_heating",
        "water_preheating",
        "greenhouse_climate_control"
      ]
    }
  }
}
```

**Measurement Requirements:**
- Thermal sensors installed at equipment exhaust points
- Temperature readings logged every 5 minutes
- Heat capture measured at point of utilization (not just at source)
- Recovery efficiency = `heat_delivered_to_offtaker / total_thermal_output`

**Exemptions:**
- Companies with total thermal load <50kW are exempt from heat recovery requirements
- Geographic locations where heat recovery is impractical (tropical climates, summer months) receive seasonal exemptions

**Data Source Integration:**
- IoT thermal sensors (Modbus, BACnet protocols)
- Building Management Systems (BMS)
- Custom sensor networks with MQTT or REST API endpoints

---

#### E. Telemetry Coverage Verification

**Purpose:** Prevent companies from cherry-picking which infrastructure to report.

**Required Data Points:**

```json
{
  "telemetry_coverage": {
    "bronze_inventory_baseline": {
      "submission_date": "2024-09-12",
      "total_instances_documented": 847,
      "total_storage_buckets_documented": 284,
      "total_model_endpoints_documented": 12
    },
    "current_telemetry_sources": {
      "instances_reporting": 831,
      "storage_reporting": 284,
      "model_endpoints_reporting": 12
    },
    "coverage_ratios": {
      "instance_coverage": 0.981,
      "storage_coverage": 1.000,
      "model_coverage": 1.000,
      "overall_coverage": 0.987
    },
    "missing_instances": [
      "i-0abc123def456 (dev-server-orphaned)",
      "i-0def789ghi012 (staging-db-deprecated)"
    ]
  }
}
```

**Enforcement:**
- Companies must maintain ≥95% coverage across all infrastructure categories
- Missing instances must be either:
  - Added to telemetry within 7 days, OR
  - Documented as decommissioned with termination logs
- New infrastructure must be added to telemetry within 48 hours of launch

---

### III. API Integration Patterns

#### A. Cloud Provider Integration (Recommended)

**AWS Implementation:**

```python
# Company grants CMS Dashboard read-only IAM role with specific permissions

IAM_POLICY = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:GetMetricStatistics",
                "cloudwatch:ListMetrics",
                "ec2:DescribeInstances",
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation",
                "s3:GetBucketInventoryConfiguration",
                "cloudtrail:LookupEvents"
            ],
            "Resource": "*"
        }
    ]
}

# CMS Dashboard assumes this role to pull metrics
# Company can revoke access at any time (which auto-revokes Obsidian status)
```

**Azure Implementation:**

```python
# Company creates Service Principal with Reader role + specific permissions

AZURE_PERMISSIONS = [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Insights/metrics/read",
    "Microsoft.Storage/storageAccounts/read",
    "Microsoft.Storage/storageAccounts/blobServices/containers/read"
]

# CMS Dashboard uses Service Principal credentials to query Azure Monitor
```

**GCP Implementation:**

```python
# Company creates custom IAM role with monitoring permissions

GCP_ROLE_PERMISSIONS = [
    "compute.instances.list",
    "monitoring.timeSeries.list",
    "storage.buckets.list",
    "storage.objects.list",
    "logging.logEntries.list"
]

# CMS Dashboard uses service account with this custom role
```

---

#### B. On-Premise / Hybrid Integration

**For companies with on-prem or hybrid infrastructure:**

**Option 1: Prometheus Integration (Recommended)**

```yaml
# Company runs Prometheus with exporters for compute, storage, and thermal metrics
# CMS Dashboard polls Prometheus /api/v1/query endpoint

prometheus_config:
  scrape_interval: 15m
  exporters:
    - node_exporter  # CPU, memory, disk
    - snmp_exporter  # Thermal sensors
    - custom_model_exporter  # AI inference logs
```

**Option 2: CMS Telemetry Agent (Open Source)**

```bash
# Company deploys lightweight agent that collects metrics and forwards to CMS Dashboard

cms-telemetry-agent install \
  --company-id "acme-corp" \
  --api-key "cms_live_1234567890abcdef" \
  --collect-compute \
  --collect-storage \
  --collect-models \
  --collect-thermal \
  --interval 15m
```

Agent source code: `github.com/carbon-mirror-standard/telemetry-agent`

---

### IV. Data Transmission and Security

#### A. Authentication

**API Key Method:**
- Company generates unique API key in CMS Dashboard
- Key is scoped to read-only telemetry submission
- Key can be rotated without disrupting certification

**OAuth 2.0 Method (Cloud Providers):**
- CMS Dashboard acts as OAuth client
- Company authorizes read-only access via cloud provider's OAuth flow
- Token refresh handled automatically

#### B. Encryption

- All telemetry data transmitted via **TLS 1.3 or higher**
- Data at rest encrypted with AES-256
- Company retains ownership of raw telemetry data
- CMS Dashboard stores only aggregated metrics and historical trends

#### C. Data Retention

- Raw telemetry retained for **90 days** (for anomaly detection)
- Aggregated metrics retained for **365 days** (for historical comparison)
- Certification events retained **indefinitely** (public audit trail)

#### D. Privacy Protections

**What CMS Dashboard DOES NOT collect:**
- Customer data or PII
- Application code or proprietary algorithms
- Business logic or trade secrets
- Detailed request contents (only metadata: task type, model used, token count)

**What IS collected:**
- Infrastructure utilization metrics
- Model routing decisions (which model for which task type)
- Storage access patterns (not file contents)
- Thermal output (not what workloads generated it)

---

### V. Telemetry Submission Schedule

#### Obsidian Real-Time Requirements

```
Metric Category          | Submission Frequency | Calculation Window
-------------------------|---------------------|-------------------
Ghost Compute            | Every 15 minutes    | Rolling 30-day avg
Model Efficiency         | Every 15 minutes    | Rolling 30-day total
Dark Data                | Every 24 hours      | Point-in-time snapshot
Heat Recovery            | Every 5 minutes     | Rolling 24-hour avg
Coverage Verification    | Every 24 hours      | Point-in-time comparison
```

#### Bandwidth Impact Estimation

For a mid-size company (1000 instances, 500TB storage, 100K daily inferences):

```
Metric transmission size per interval:
- Ghost Compute: ~150KB per submission (instance metadata)
- Model Efficiency: ~80KB per submission (inference log aggregates)
- Dark Data: ~2MB per submission (storage inventory)
- Heat Recovery: ~10KB per submission (sensor readings)

Total daily bandwidth: ~250MB/day
Total monthly bandwidth: ~7.5GB/month
```

**This is negligible** compared to typical datacenter network usage.

---

### VI. Telemetry API Endpoints (CMS Dashboard)

Companies submit telemetry to these standardized endpoints:

#### Submission Endpoints

```
POST /api/v1/telemetry/ghost-compute
POST /api/v1/telemetry/model-efficiency  
POST /api/v1/telemetry/dark-data
POST /api/v1/telemetry/heat-recovery
POST /api/v1/telemetry/coverage-verification
```

#### Query Endpoints (Public)

```
GET /api/v1/company/{id}/metrics/current
GET /api/v1/company/{id}/metrics/history?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD
GET /api/v1/company/{id}/status
```

#### Webhook Notifications (Optional)

Companies can register webhooks to receive alerts:

```json
{
  "webhook_url": "https://company.com/cms-notifications",
  "events": [
    "metric_warning",
    "grace_period_started", 
    "certification_revoked",
    "certification_restored"
  ]
}
```

---

### VII. Reference Implementation

**Minimal Python Example (AWS):**

```python
import boto3
import requests
from datetime import datetime, timedelta

class CMSTelemetrySubmitter:
    def __init__(self, company_id, api_key):
        self.company_id = company_id
        self.api_key = api_key
        self.cms_endpoint = "https://dashboard.carbonmirror.org/api/v1"
        self.cloudwatch = boto3.client('cloudwatch')
        self.ec2 = boto3.client('ec2')
    
    def collect_ghost_compute_metrics(self):
        """Collect CPU utilization for all running instances"""
        instances = self.ec2.describe_instances(
            Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
        )
        
        ghost_instances = []
        
        for reservation in instances['Reservations']:
            for instance in reservation['Instances']:
                instance_id = instance['InstanceId']
                
                # Get 30-day average CPU utilization
                metrics = self.cloudwatch.get_metric_statistics(
                    Namespace='AWS/EC2',
                    MetricName='CPUUtilization',
                    Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
                    StartTime=datetime.utcnow() - timedelta(days=30),
                    EndTime=datetime.utcnow(),
                    Period=86400,  # Daily aggregation
                    Statistics=['Average']
                )
                
                avg_cpu = sum(m['Average'] for m in metrics['Datapoints']) / len(metrics['Datapoints'])
                
                if avg_cpu < 10.0:
                    ghost_instances.append({
                        'instance_id': instance_id,
                        'instance_type': instance['InstanceType'],
                        'cpu_utilization_30day_avg': round(avg_cpu, 2)
                    })
        
        return ghost_instances
    
    def submit_telemetry(self, metric_type, data):
        """Submit telemetry to CMS Dashboard"""
        response = requests.post(
            f"{self.cms_endpoint}/telemetry/{metric_type}",
            headers={
                'Authorization': f'Bearer {self.api_key}',
                'Content-Type': 'application/json'
            },
            json={
                'company_id': self.company_id,
                'timestamp': datetime.utcnow().isoformat(),
                'data': data
            }
        )
        
        return response.json()

# Usage
submitter = CMSTelemetrySubmitter(
    company_id='acme-corp',
    api_key='cms_live_abc123'
)

ghost_data = submitter.collect_ghost_compute_metrics()
result = submitter.submit_telemetry('ghost-compute', ghost_data)
print(f"Submitted {len(ghost_data)} ghost instances. Status: {result['status']}")
```

Full reference implementations available at:
- `github.com/carbon-mirror-standard/telemetry-examples/aws`
- `github.com/carbon-mirror-standard/telemetry-examples/azure`
- `github.com/carbon-mirror-standard/telemetry-examples/gcp`
- `github.com/carbon-mirror-standard/telemetry-examples/on-prem`

---

### VIII. Compliance and Auditing

#### Telemetry Validation

CMS Dashboard performs continuous validation:

```python
def validate_telemetry_submission(company_id, submission):
    """
    Verify submitted telemetry meets quality standards
    """
    
    # Check 1: Timestamp freshness
    if submission.age_minutes > 30:
        return {"valid": False, "reason": "Stale data (>30min old)"}
    
    # Check 2: Data completeness
    required_fields = get_required_fields(submission.metric_type)
    if not all(field in submission.data for field in required_fields):
        return {"valid": False, "reason": "Missing required fields"}
    
    # Check 3: Statistical plausibility
    if detect_anomaly(company_id, submission):
        flag_for_review(company_id, submission)
        return {"valid": True, "warning": "Anomaly detected - under review"}
    
    return {"valid": True}
```

#### Spot-Check Audits

- CMS Foundation performs **random spot-checks** on 2% of Obsidian-certified companies monthly
- Spot-check involves independent verification of telemetry accuracy
- If discrepancies found, company placed on probation for 30 days
- Repeated discrepancies result in permanent Obsidian ineligibility

---

### IX. Migration Path for New Adopters

**Phase 1: Read-Only Integration (Weeks 1-2)**
- Grant CMS Dashboard API access
- Verify telemetry collection works
- Review initial metrics (no certification impact)

**Phase 2: Baseline Establishment (Weeks 3-6)**
- CMS Dashboard builds 30-day historical baseline
- Company identifies areas needing improvement
- No public status published yet

**Phase 3: Obsidian Certification Attempt (Week 7+)**
- Company opts in to public certification evaluation
- If metrics meet thresholds, Obsidian status granted
- If not, company remains Silver until fixes implemented

**This gradual onboarding prevents "day-one revocation" scenarios.**

---

### X. Future Extensions

**Planned for v2.0:**
- Carbon intensity-aware scheduling telemetry (workload timing vs grid carbon)
- Network efficiency metrics (data transfer optimization)
- Container/Kubernetes-specific telemetry
- Edge computing and CDN efficiency tracking
- Renewable energy usage correlation

---

### XI. Open Source Commitment

**All telemetry standards, schemas, and reference implementations are open source.**

- License: Apache 2.0
- Repository: `github.com/carbon-mirror-standard/telemetry-spec`
- Community contributions welcome via pull requests
- Breaking changes require 90-day deprecation notice

**The telemetry protocol itself is vendor-neutral and cannot be owned by any single company.**

---

*Carbon Mirror Standard Telemetry API Specification v1.0*  
*Published: [Date]*  
*Maintained by: CMS Working Group*  
*Feedback: telemetry-feedback@carbonmirror.org*  
*License: Apache 2.0*
