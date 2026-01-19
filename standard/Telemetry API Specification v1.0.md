# Carbon Mirror Standard
## Telemetry API Specification v1.0

### I. Purpose and Scope

This document defines the technical requirements for companies seeking Obsidian Seal certification under the Carbon Mirror Standard. It specifies the exact telemetry data that must be exposed, the format in which it must be delivered, and the integration mechanisms for major cloud providers and on-premise infrastructure.

**Core Principle:** Obsidian certification requires **continuous, unforgeable telemetry** that the CMS Dashboard can verify independently. Companies cannot self-report metrics—data must be pulled directly from infrastructure monitoring systems.

---

### II. Telemetry Data Requirements

#### A. Ghost Compute Metrics

**Purpose:** Measure wasted energy consumption from idling infrastructure relative to total productive energy use.

**Core Principle:** Ghost Compute is measured as **wasted energy as a percentage of total energy consumption**, not simply idle instance count. This ensures fair comparison across organizations of different scales—a startup with 10 servers and a hyperscaler with 100,000 servers are judged by energy efficiency per unit of productive work.

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
        "power_draw_watts": 285,
        "classification": "ghost",
        "monthly_cost_usd": 280.32,
        "monthly_energy_kwh": 205.2,
        "monthly_energy_cost_usd": 20.52
      }
    ],
    "energy_summary": {
      "total_instances": 847,
      "ghost_instances": 34,
      "productive_instances": 813,
      "ghost_energy_kwh_daily": 232.8,
      "productive_energy_kwh_daily": 4891.2,
      "total_energy_kwh_daily": 5124.0,
      "ghost_energy_ratio": 0.0454,
      "ghost_efficiency_score": 95.46,
      "pue_multiplier": 1.42,
      "total_energy_with_cooling_kwh_daily": 7276.1
    },
    "physical_impact": {
      "monthly_waste_usd": 9530.88,
      "co2e_kg_monthly": 5890,
      "water_liters_monthly": 12970,
      "component_stress_instances": 34,
      "grid_capacity_wasted_mw": 0.245
    }
  }
}
```

**Calculation Rules:**

**Ghost Instance Classification:**
- Instance classified as "ghost" if `cpu_utilization_30day_avg < 10%` AND `memory_utilization_30day_avg < 10%`
- Metrics calculated as rolling 30-day average, updated every 15 minutes
- All running instances must be reported (no cherry-picking)

**Energy-Based Efficiency Score:**
```python
# Calculate wasted energy
ghost_energy_kwh = sum(instance.power_draw_kw * 24 for instance in ghost_instances)

# Calculate productive energy  
productive_energy_kwh = sum(instance.power_draw_kw * 24 for instance in productive_instances)

# Total energy consumption
total_energy_kwh = ghost_energy_kwh + productive_energy_kwh

# Ghost energy ratio (waste percentage)
ghost_energy_ratio = ghost_energy_kwh / total_energy_kwh

# Efficiency score (inverted - higher is better)
ghost_efficiency_score = (1 - ghost_energy_ratio) * 100

# Score brackets:
# 98%+ (≤2% waste) = Apex tier contribution
# 95-97.9% (2-5% waste) = Prime tier contribution  
# 90-94.9% (5-10% waste) = Core tier contribution
# 85-89.9% (10-15% waste) = Base Obsidian contribution
# <85% (>15% waste) = Below Obsidian threshold
```

**PUE (Power Usage Effectiveness) Integration:**
Total energy impact includes cooling overhead:
```python
total_facility_energy = total_energy_kwh * pue_multiplier
# Industry average PUE: 1.4-1.6
# Efficient datacenters: 1.2-1.3  
# Best-in-class: <1.2
```

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

### III. Composite Efficiency Scoring and Tier Certification

#### A. Composite Efficiency Calculation

Obsidian certification status is determined by a **weighted composite score** across all four efficiency metrics. This prevents companies from gaming individual metrics while ignoring others.

**Scoring Formula:**

```python
def calculate_obsidian_composite_score(company_data):
    """
    Calculate overall efficiency score for tier certification.
    All four metrics must meet minimum thresholds.
    """
    
    # Metric 1: Ghost Compute Efficiency (energy-based)
    ghost_score = calculate_ghost_efficiency_score(company_data.ghost_compute)
    
    # Metric 2: Model Efficiency (routine-to-SLM ratio)
    model_score = calculate_model_efficiency_score(company_data.model_efficiency)
    
    # Metric 3: Dark Data Efficiency (storage utilization)
    storage_score = calculate_dark_data_efficiency_score(company_data.dark_data)
    
    # Metric 4: Heat Recovery (if thermal load ≥50kW, otherwise exempt)
    if company_data.thermal_load_kw >= 50:
        heat_score = calculate_heat_recovery_score(company_data.heat_recovery)
        weights = [0.25, 0.25, 0.25, 0.25]
        scores = [ghost_score, model_score, storage_score, heat_score]
    else:
        # Small deployments: heat recovery exempt
        weights = [0.33, 0.33, 0.34]
        scores = [ghost_score, model_score, storage_score]
    
    # Weighted average
    composite_score = sum(score * weight for score, weight in zip(scores, weights))
    
    # Individual metric minimums (prevent single-metric gaming)
    min_acceptable = 75  # No metric can fall below 75% efficiency
    
    if any(score < min_acceptable for score in scores):
        return {
            "composite_score": composite_score,
            "tier": "Silver",
            "reason": f"At least one metric below {min_acceptable}% threshold",
            "failing_metrics": [name for name, score in zip(
                ["ghost_compute", "model_efficiency", "dark_data", "heat_recovery"][:len(scores)], 
                scores
            ) if score < min_acceptable]
        }
    
    # Determine tier based on composite score
    return {
        "composite_score": round(composite_score, 2),
        "tier": get_tier_from_score(composite_score),
        "individual_scores": {
            "ghost_compute": ghost_score,
            "model_efficiency": model_score,
            "dark_data": storage_score,
            "heat_recovery": heat_score if company_data.thermal_load_kw >= 50 else "exempt"
        }
    }

def get_tier_from_score(composite_score):
    """Map composite efficiency score to certification tier"""
    if composite_score >= 95.0:
        return "Obsidian Apex"
    elif composite_score >= 90.0:
        return "Obsidian Prime"
    elif composite_score >= 85.0:
        return "Obsidian Core"
    elif composite_score >= 80.0:
        return "Obsidian"
    else:
        return "Silver"  # Below Obsidian threshold
```

#### B. Certification Tier Structure

**Obsidian Tiers** (Real-time telemetry required):

| Tier | Composite Score | Description | Typical Profile |
|------|----------------|-------------|-----------------|
| **Obsidian Apex** | 95.0%+ | Elite efficiency - best-in-class operations | <3% of certified orgs |
| **Obsidian Prime** | 90.0-94.9% | High efficiency - strong operational discipline | ~12% of certified orgs |
| **Obsidian Core** | 85.0-89.9% | Above-average efficiency - solid performance | ~35% of certified orgs |
| **Obsidian** | 80.0-84.9% | Baseline efficiency - meets minimum standard | ~50% of certified orgs |
| **Silver** | <80.0% | Pre-Obsidian - has accountability but not yet optimized | - |
| **Bronze** | N/A | Completed Friction Audit (one-time achievement) | - |

**Tier Transition Rules:**

```python
def monitor_tier_transitions(company_id):
    """
    Real-time monitoring of tier status with 7-day grace period
    for downward transitions, immediate promotion for upward.
    """
    
    current_tier = get_current_tier(company_id)
    current_score = calculate_obsidian_composite_score(company_id)
    new_tier = current_score["tier"]
    
    if new_tier == current_tier:
        # No change, reset any grace period counters
        reset_grace_period(company_id)
        return
    
    # UPWARD TRANSITION: Immediate promotion
    if is_higher_tier(new_tier, current_tier):
        promote_tier(
            company_id=company_id,
            from_tier=current_tier,
            to_tier=new_tier,
            timestamp=current_timestamp(),
            reason="Composite efficiency improved"
        )
        
        publish_promotion_event(company_id, current_tier, new_tier)
        reset_grace_period(company_id)
        return
    
    # DOWNWARD TRANSITION: 7-day grace period
    if is_lower_tier(new_tier, current_tier):
        grace_period_days = increment_grace_period(company_id)
        
        if grace_period_days < 7:
            publish_warning(
                company_id=company_id,
                message=f"Efficiency below {current_tier} threshold. "
                        f"Revocation to {new_tier} in {7 - grace_period_days} days if not resolved.",
                days_remaining=7 - grace_period_days
            )
        else:
            # Grace period expired
            revoke_tier(
                company_id=company_id,
                from_tier=current_tier,
                to_tier=new_tier,
                timestamp=current_timestamp(),
                reason=f"Composite efficiency below {current_tier} threshold for 7 consecutive days"
            )
            
            publish_revocation_event(company_id, current_tier, new_tier, current_score)
            reset_grace_period(company_id)
```

#### C. Public Dashboard Display

**Example Company Status:**

```json
{
  "company_name": "Example Corp",
  "company_id": "examplecorp",
  "certification_tier": "Obsidian Prime",
  "composite_efficiency": 92.3,
  "last_updated": "2026-01-16T15:45:00Z",
  "individual_metrics": {
    "ghost_compute": {
      "efficiency_score": 94.2,
      "ghost_energy_ratio": 0.058,
      "wasted_kwh_daily": 342,
      "tier_contribution": "Prime"
    },
    "model_efficiency": {
      "efficiency_score": 91.8,
      "routine_to_slm_ratio": 0.918,
      "tier_contribution": "Prime"
    },
    "dark_data": {
      "efficiency_score": 89.7,
      "dark_data_ratio": 0.103,
      "tier_contribution": "Core"
    },
    "heat_recovery": {
      "efficiency_score": 93.5,
      "recovery_ratio": 0.935,
      "tier_contribution": "Prime"
    }
  },
  "physical_impact": {
    "financial_waste_monthly_usd": 47200,
    "co2e_monthly_kg": 28400,
    "water_monthly_liters": 89000,
    "grid_capacity_wasted_mw": 1.2,
    "equivalent_homes": 960
  },
  "tier_history": [
    {"date": "2025-11-03", "tier": "Obsidian Core", "score": 87.1},
    {"date": "2025-12-18", "tier": "Obsidian Prime", "score": 90.4},
    {"date": "2026-01-16", "tier": "Obsidian Prime", "score": 92.3}
  ],
  "grid_citizen": false
}
```

#### D. Grid Emergency Response (Optional Enhancement)

Companies can earn **Grid Citizen** designation by implementing automated emergency response protocols.

**Requirements for Grid Citizen Status:**

1. **Real-time grid operator API integration**
2. **Automated load shedding** during declared emergencies (Stage 1+)
3. **50%+ ghost compute reduction** within 15 minutes of emergency declaration
4. **Documented recovery procedures** after emergency cleared
5. **Successful emergency response** in 100% of events in past 12 months

**Grid Emergency Response Verification:**

```python
def verify_grid_citizen_status(company_id):
    """
    Validate that company responds appropriately to grid emergencies.
    Failure results in immediate Grid Citizen revocation.
    """
    
    grid_region = get_company_grid_region(company_id)
    emergency_events = get_grid_emergencies(grid_region, period="12months")
    
    successful_responses = 0
    failed_responses = []
    
    for event in emergency_events:
        if event.severity >= "Stage 1":
            # Measure ghost compute before and during emergency
            ghost_before = get_ghost_compute_mw(company_id, event.start_time - timedelta(hours=1))
            ghost_during = get_ghost_compute_mw(company_id, event.start_time + timedelta(minutes=30))
            
            reduction_pct = (ghost_before - ghost_during) / ghost_before if ghost_before > 0 else 0
            
            if reduction_pct >= 0.50:  # Met 50% reduction requirement
                successful_responses += 1
            else:
                failed_responses.append({
                    "event": event.name,
                    "date": event.start_time,
                    "required_reduction": 0.50,
                    "actual_reduction": reduction_pct,
                    "capacity_withheld_mw": ghost_during
                })
    
    if len(emergency_events) == 0:
        # No emergencies in region during period - maintain status if already certified
        return {"grid_citizen": get_current_grid_citizen_status(company_id), "reason": "No grid emergencies in past 12 months"}
    
    success_rate = successful_responses / len(emergency_events)
    
    if success_rate == 1.0:
        return {
            "grid_citizen": True,
            "events_responded": successful_responses,
            "total_events": len(emergency_events),
            "success_rate": 1.0
        }
    else:
        return {
            "grid_citizen": False,
            "events_responded": successful_responses,
            "total_events": len(emergency_events),
            "success_rate": success_rate,
            "failed_responses": failed_responses,
            "reason": "Did not achieve 100% emergency response success rate"
        }
```

**Grid Citizen Public Display:**

```json
{
  "company_name": "Responsible Corp",
  "certification_tier": "Obsidian Apex",
  "grid_citizen": true,
  "grid_citizen_details": {
    "emergency_response": "Automated",
    "last_test_date": "2025-12-08",
    "test_result": "Passed (shed 2.4 MW in 12 minutes)",
    "events_responded": 4,
    "total_events": 4,
    "success_rate": 1.0,
    "public_benefit_homes_equivalent": 1920
  }
}
```

**Immediate Revocation for Emergency Non-Compliance:**

If a Grid Citizen certified company fails to respond appropriately during a grid emergency:

```python
if grid_emergency_detected and grid_citizen_status and reduction_pct < 0.50:
    revoke_immediately(
        company_id=company_id,
        from_tier=current_tier,
        to_tier="Silver",
        revocation_type="EMERGENCY_NON_COMPLIANCE",
        restoration_blocked_days=90,
        public_notice=f"Failed to shed ghost compute during {event.name} grid emergency. "
                     f"Required: 50% reduction. Actual: {reduction_pct:.1%}. "
                     f"Capacity withheld: {ghost_during:.2f} MW (equivalent to {homes_equivalent} homes)."
    )
```

---

### IV. Shared Infrastructure Impact Reporting

In addition to individual efficiency metrics, the CMS Dashboard publishes the **Shared Infrastructure Impact** for each certified company. This makes visible how operational waste degrades service quality for other users of shared digital resources.

**Shared Infrastructure Impact Categories:**

```json
{
  "shared_infrastructure_impact": {
    "network_waste": {
      "unnecessary_api_calls_daily": 2400000,
      "bandwidth_consumed_monthly_tb": 180,
      "peak_hour_traffic_percentage": 0.45,
      "equivalent_remote_users_denied": 12000,
      "assessment": "High impact - significant peak-hour congestion contribution"
    },
    "storage_waste": {
      "dark_data_volume_tb": 840,
      "backup_bandwidth_daily_tb": 6.2,
      "network_stress_level": "Moderate",
      "assessment": "Moderate impact - backup operations contribute to backbone congestion"
    },
    "grid_impact": {
      "wasted_grid_draw_mw": 1.8,
      "peak_contribution_mw": 2.4,
      "equivalent_homes": 1440,
      "equivalent_hospitals": 3,
      "equivalent_emergency_dispatch_centers": 12,
      "grid_citizen_status": false,
      "assessment": "Significant impact - wasted capacity could serve critical infrastructure"
    },
    "public_summary": "Operational waste equivalent to denying service to 12,000 remote users, consuming grid capacity for 1,440 homes, and contributing to network congestion during peak hours."
  }
}
```

**This data is prominently displayed but not directly scored**, as it is a consequence of the efficiency metrics rather than a separate measurement. However, making it visible transforms corporate efficiency into a public good story.

---

### V. API Integration Patterns

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

### VI. Data Transmission and Security

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

### VII. Telemetry Submission Schedule

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

### VIII. Telemetry API Endpoints (CMS Dashboard)

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

### IX. Reference Implementation

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
    
    def get_instance_power_draw(self, instance_type):
        """
        Estimate power draw based on instance type.
        Real implementations should use actual power monitoring.
        """
        # Simplified power draw estimates (watts)
        power_map = {
            't2.micro': 10, 't2.small': 15, 't2.medium': 20,
            'm5.large': 85, 'm5.xlarge': 140, 'm5.2xlarge': 285,
            'c5.large': 95, 'c5.xlarge': 190, 'c5.2xlarge': 380,
            'r5.large': 90, 'r5.xlarge': 180, 'r5.2xlarge': 360,
            # Add more as needed
        }
        return power_map.get(instance_type, 100)  # Default 100W if unknown
    
    def collect_ghost_compute_metrics(self):
        """
        Collect energy-based ghost compute metrics for all running instances
        """
        instances = self.ec2.describe_instances(
            Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
        )
        
        ghost_instances = []
        productive_instances = []
        ghost_energy_kwh_daily = 0
        productive_energy_kwh_daily = 0
        
        for reservation in instances['Reservations']:
            for instance in reservation['Instances']:
                instance_id = instance['InstanceId']
                instance_type = instance['InstanceType']
                
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
                
                if len(metrics['Datapoints']) == 0:
                    continue
                
                avg_cpu = sum(m['Average'] for m in metrics['Datapoints']) / len(metrics['Datapoints'])
                
                # Estimate power draw
                power_watts = self.get_instance_power_draw(instance_type)
                energy_kwh_daily = (power_watts / 1000) * 24
                
                instance_data = {
                    'instance_id': instance_id,
                    'instance_type': instance_type,
                    'cpu_utilization_30day_avg': round(avg_cpu, 2),
                    'power_draw_watts': power_watts,
                    'energy_kwh_daily': round(energy_kwh_daily, 3)
                }
                
                if avg_cpu < 10.0:
                    # Ghost instance
                    ghost_instances.append(instance_data)
                    ghost_energy_kwh_daily += energy_kwh_daily
                else:
                    # Productive instance
                    productive_instances.append(instance_data)
                    productive_energy_kwh_daily += energy_kwh_daily
        
        # Calculate energy-based efficiency metrics
        total_energy = ghost_energy_kwh_daily + productive_energy_kwh_daily
        ghost_energy_ratio = ghost_energy_kwh_daily / total_energy if total_energy > 0 else 0
        ghost_efficiency_score = (1 - ghost_energy_ratio) * 100
        
        return {
            'ghost_instances': ghost_instances,
            'productive_instances': productive_instances,
            'energy_summary': {
                'total_instances': len(ghost_instances) + len(productive_instances),
                'ghost_instances': len(ghost_instances),
                'productive_instances': len(productive_instances),
                'ghost_energy_kwh_daily': round(ghost_energy_kwh_daily, 2),
                'productive_energy_kwh_daily': round(productive_energy_kwh_daily, 2),
                'total_energy_kwh_daily': round(total_energy, 2),
                'ghost_energy_ratio': round(ghost_energy_ratio, 4),
                'ghost_efficiency_score': round(ghost_efficiency_score, 2)
            }
        }
    
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

print(f"Ghost Compute Telemetry Submitted:")
print(f"  Ghost Energy Ratio: {ghost_data['energy_summary']['ghost_energy_ratio']:.2%}")
print(f"  Efficiency Score: {ghost_data['energy_summary']['ghost_efficiency_score']:.1f}%")
print(f"  Status: {result['status']}")
```

Full reference implementations available at:
- `github.com/carbon-mirror-standard/telemetry-examples/aws`
- `github.com/carbon-mirror-standard/telemetry-examples/azure`
- `github.com/carbon-mirror-standard/telemetry-examples/gcp`
- `github.com/carbon-mirror-standard/telemetry-examples/on-prem`

---

### X. Compliance and Auditing

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

### XI. Migration Path for New Adopters

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

### XII. Future Extensions

**Planned for v2.0:**
- Carbon intensity-aware scheduling telemetry (workload timing vs grid carbon)
- Network efficiency metrics (data transfer optimization)
- Container/Kubernetes-specific telemetry
- Edge computing and CDN efficiency tracking
- Renewable energy usage correlation

---

### XIII. Open Source Commitment

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
