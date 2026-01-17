# Carbon Mirror Standard
## Auto-Certification Logic Specification v1.0

### I. Purpose and Authority

This document defines the deterministic algorithms that govern certification status in the Carbon Mirror Standard (CMS). These rules replace human judgment for Silver and Obsidian tiers, ensuring that certification is based on verifiable performance metrics rather than subjective assessment.

**Core Principle:** The code is the authority. No appeals, no exceptions, no human override.

---

### II. Certification Tiers and Requirements

#### Bronze Reflection (Manual Gate)

**Entry Requirements:**
- Completed Friction Audit evidence packet submitted
- Minimum 3 independent volunteer reviewer approvals
- "No-Sunshine Sign-Off" declaration signed by Lead Engineer
- Evidence must include:
  - Ghost Compute Inventory (minimum 3 specific assets documented)
  - Model Mismatch Log (minimum 1 routine task identified for SLM migration)
  - Dark Data Identification (volume estimate + disposal plan)
  - Thermal Recognition Scan (waste heat map + off-taker identification)
  - Small-First Policy documentation
  - Grid Shift Log (minimum 1 workload rescheduled to low-carbon window)

**Certification Process:**
```
IF submission_complete == TRUE
   AND reviewer_approvals >= 3
   AND sign_off_present == TRUE
THEN
   status = "Bronze Certified"
   certification_date = current_timestamp
   public_registry_update(company_id, "Bronze", certification_date)
```

**Revocation Conditions:**
- None. Bronze certification is permanent once achieved.
- Rationale: Bronze proves the company completed the foundational audit. This historical achievement doesn't expire.

---

#### Silver Intake (Semi-Automated)

**Entry Requirements:**
- Current Bronze certification
- Internal Carbon Tax structure implemented and publicly documented
- Department scoreboard accessible via public API or granted read access
- Minimum 3 consecutive months of reporting

**Certification Criteria:**

```python
def evaluate_silver_eligibility(company_data):
    """
    Evaluates Silver certification based on carbon tax implementation
    and public accountability mechanisms.
    """
    
    # Check 1: Carbon Tax Structure
    tax_documented = company_data.carbon_tax_structure_public == True
    tax_rate_visible = company_data.carbon_tax_rate_usd_per_ton != None
    
    # Check 2: Department Scoreboard Variance
    # If all departments show identical scores, the tax isn't creating real accountability
    dept_scores = company_data.department_carbon_scores
    score_variance = calculate_variance(dept_scores)
    meaningful_variance = score_variance > 0.15  # At least 15% variance required
    
    # Check 3: Reporting Consistency
    reports_on_time = company_data.monthly_reports_submitted >= 3
    no_gaps = company_data.reporting_gaps == 0
    
    # Certification Logic
    if (tax_documented and 
        tax_rate_visible and 
        meaningful_variance and 
        reports_on_time and 
        no_gaps):
        return "Silver Certified"
    else:
        return "Bronze" # Remain at Bronze if any check fails
```

**Auto-Revocation Triggers:**

```python
def check_silver_revocation(company_id):
    """
    Silver certification revokes automatically if accountability mechanisms lapse.
    """
    
    # Trigger 1: Missed Reporting
    if days_since_last_report(company_id) > 45:
        revoke_to_bronze(company_id, reason="Reporting lapsed")
        return True
    
    # Trigger 2: Variance Collapse (tax not creating accountability)
    if department_variance(company_id) < 0.10 for 2_consecutive_months:
        revoke_to_bronze(company_id, reason="Scoreboard variance insufficient")
        return True
    
    # Trigger 3: Tax Structure Hidden
    if carbon_tax_documentation_public(company_id) == False:
        revoke_to_bronze(company_id, reason="Tax structure no longer public")
        return True
    
    return False # Certification maintained
```

**Restoration:**
- Automatic upon meeting criteria again for 2 consecutive months
- No waiting period, no reapplication required

---

#### Obsidian Seal (Fully Automated)

**Entry Requirements:**
- Current Silver certification
- Live telemetry API access granted to CMS Dashboard
- Telemetry must cover ≥95% of infrastructure inventory by instance count
- Minimum 30-day telemetry history before initial evaluation

**Real-Time Certification Thresholds:**

```python
OBSIDIAN_THRESHOLDS = {
    "ghost_compute_max": 0.05,        # ≤5% of instances idling
    "model_efficiency_min": 0.70,     # ≥70% routine tasks to SLM
    "dark_data_max": 0.15,            # ≤15% of storage unused 180+ days
    "heat_recovery_min": 0.25,        # ≥25% waste heat captured (if applicable)
    "telemetry_coverage_min": 0.95    # ≥95% infrastructure reporting
}

def calculate_obsidian_status(company_id):
    """
    Live calculation every 15 minutes.
    Uses rolling 30-day averages to prevent gaming via temporary fixes.
    """
    
    metrics = get_rolling_30day_metrics(company_id)
    
    # Metric 1: Ghost Compute Ratio
    total_instances = metrics.total_instance_count
    idle_instances = count_instances_where(utilization < 0.10, period="30day")
    ghost_ratio = idle_instances / total_instances
    
    # Metric 2: Model Efficiency Ratio
    routine_tasks = count_tasks_classified_as("routine", period="30day")
    routine_to_slm = count_tasks_where(
        task_type="routine" AND model_params <= 30_billion,
        period="30day"
    )
    model_efficiency = routine_to_slm / routine_tasks if routine_tasks > 0 else 0
    
    # Metric 3: Dark Data Ratio
    total_storage_gb = metrics.total_storage_capacity
    dark_data_gb = sum_storage_where(days_since_access > 180)
    dark_ratio = dark_data_gb / total_storage_gb
    
    # Metric 4: Heat Recovery Ratio (conditional)
    if metrics.thermal_load_kw >= 50:  # Only required if significant thermal output
        captured_heat_kw = metrics.heat_recovery_output_kw
        waste_heat_kw = metrics.total_thermal_output_kw
        heat_recovery_ratio = captured_heat_kw / waste_heat_kw
        heat_check = heat_recovery_ratio >= OBSIDIAN_THRESHOLDS["heat_recovery_min"]
    else:
        heat_check = True  # Exempt if thermal load is negligible
    
    # Metric 5: Telemetry Coverage
    expected_instances = get_bronze_inventory_count(company_id)
    reporting_instances = metrics.active_telemetry_sources
    coverage_ratio = reporting_instances / expected_instances
    
    # Certification Decision
    if (ghost_ratio <= OBSIDIAN_THRESHOLDS["ghost_compute_max"] and
        model_efficiency >= OBSIDIAN_THRESHOLDS["model_efficiency_min"] and
        dark_ratio <= OBSIDIAN_THRESHOLDS["dark_data_max"] and
        heat_check == True and
        coverage_ratio >= OBSIDIAN_THRESHOLDS["telemetry_coverage_min"]):
        
        return {
            "status": "Obsidian Certified",
            "timestamp": current_timestamp(),
            "metrics": {
                "ghost_compute": round(ghost_ratio * 100, 2),
                "model_efficiency": round(model_efficiency * 100, 2),
                "dark_data": round(dark_ratio * 100, 2),
                "heat_recovery": round(heat_recovery_ratio * 100, 2) if heat_check else "N/A",
                "telemetry_coverage": round(coverage_ratio * 100, 2)
            }
        }
    else:
        return {
            "status": "Silver",
            "reason": identify_failing_metrics(metrics),
            "timestamp": current_timestamp()
        }
```

**Auto-Revocation Logic:**

```python
def monitor_obsidian_revocation(company_id):
    """
    Runs every 15 minutes. Tracks consecutive failure days.
    Revokes to Silver after 7 consecutive days of any metric failure.
    """
    
    current_metrics = calculate_obsidian_status(company_id)
    
    if current_metrics["status"] == "Silver":  # At least one metric failing
        
        # Increment failure counter
        failure_streak = increment_failure_streak(company_id)
        
        if failure_streak >= 7 * 96:  # 7 days * 96 checks per day (15-min intervals)
            
            revoke_certification(
                company_id=company_id,
                from_tier="Obsidian",
                to_tier="Silver",
                reason=current_metrics["reason"],
                timestamp=current_timestamp()
            )
            
            publish_revocation_event(
                company_id=company_id,
                failed_metrics=current_metrics["reason"],
                revocation_date=current_date()
            )
            
            reset_failure_streak(company_id)
            
    else:  # All metrics passing
        reset_failure_streak(company_id)
        
        # Auto-restore if company was previously revoked
        if get_current_status(company_id) == "Silver" and get_previous_status(company_id) == "Obsidian":
            restore_certification(
                company_id=company_id,
                to_tier="Obsidian",
                timestamp=current_timestamp()
            )
            
            publish_restoration_event(
                company_id=company_id,
                restoration_date=current_date()
            )
```

---

### III. Anti-Gaming Mechanisms

#### Synthetic Load Detection

```python
def detect_synthetic_workload(company_id, instance_id):
    """
    Flags instances where utilization spikes don't correlate with business activity.
    """
    
    cpu_utilization = get_metric(instance_id, "cpu_avg", period="24h")
    network_io = get_metric(instance_id, "network_throughput", period="24h")
    disk_io = get_metric(instance_id, "disk_operations", period="24h")
    
    # Check for utilization spike without corresponding I/O activity
    if cpu_utilization > 0.60 and (network_io < 1_MB_per_sec or disk_io < 100_ops_per_sec):
        flag_anomaly(
            company_id=company_id,
            instance_id=instance_id,
            anomaly_type="Suspected synthetic load",
            severity="WARNING"
        )
        
        # If pattern persists for 48 hours, exclude instance from Obsidian calculations
        if anomaly_duration(instance_id) > 48_hours:
            exclude_from_certification_metrics(instance_id)
```

#### Telemetry Coverage Enforcement

```python
def enforce_telemetry_coverage(company_id):
    """
    Companies cannot cherry-pick which infrastructure to report.
    Bronze inventory serves as the baseline - Obsidian must cover 95%+ of it.
    """
    
    bronze_inventory = get_bronze_submitted_inventory(company_id)
    current_telemetry_sources = get_active_telemetry_instances(company_id)
    
    coverage_ratio = len(current_telemetry_sources) / len(bronze_inventory)
    
    if coverage_ratio < OBSIDIAN_THRESHOLDS["telemetry_coverage_min"]:
        return {
            "eligible": False,
            "reason": f"Telemetry coverage {coverage_ratio:.1%} below required 95%",
            "missing_instances": bronze_inventory - current_telemetry_sources
        }
    else:
        return {"eligible": True}
```

#### Data Reclassification Detection

```python
def detect_dark_data_reclassification(company_id):
    """
    Prevents gaming by suddenly 'accessing' old data to reset its age counter.
    """
    
    current_dark_data = get_dark_data_volume(company_id, cutoff_days=180)
    historical_dark_data = get_metric_history(company_id, "dark_data_volume", period="90day")
    
    # Check for sudden, massive drops in dark data volume
    if len(historical_dark_data) >= 30:  # Need at least 30 days of history
        avg_historical = mean(historical_dark_data[-30:])
        
        drop_percentage = (avg_historical - current_dark_data) / avg_historical
        
        # Flag if >30% of dark data suddenly "disappears" without deletion logs
        if drop_percentage > 0.30:
            deletion_events = get_storage_deletion_logs(company_id, period="7day")
            
            if deletion_events.total_deleted_gb < (avg_historical * 0.25):
                flag_anomaly(
                    company_id=company_id,
                    anomaly_type="Suspected dark data reclassification without deletion",
                    severity="CRITICAL",
                    details=f"Dark data dropped {drop_percentage:.1%} but deletion logs only show {deletion_events.total_deleted_gb}GB removed"
                )
                
                # Use historical average instead of current value for certification
                override_metric(company_id, "dark_data_volume", avg_historical)
```

---

### IV. Public Registry and Transparency

#### Real-Time Status Publication

```python
def publish_certification_status(company_id):
    """
    Every company's status is publicly queryable in real-time.
    No delay, no embargo period.
    """
    
    return {
        "company_name": get_company_name(company_id),
        "current_tier": get_current_tier(company_id),
        "certification_date": get_certification_date(company_id),
        "last_status_change": get_last_change_timestamp(company_id),
        "metrics_snapshot": get_latest_metrics(company_id) if tier == "Obsidian" else None,
        "certification_history": get_tier_history(company_id),
        "revocation_events": get_revocation_log(company_id)
    }
```

#### Historical Audit Trail

All certification changes are permanently logged:

```python
CERTIFICATION_EVENT_LOG = {
    "event_id": uuid,
    "company_id": string,
    "timestamp": datetime,
    "event_type": ["certification", "revocation", "restoration"],
    "from_tier": string,
    "to_tier": string,
    "trigger_reason": string,
    "metrics_at_event": dict,
    "automated": boolean  # True for Silver/Obsidian, False for Bronze manual review
}
```

This log is:
- **Immutable** (append-only, no deletions)
- **Public** (anyone can query any company's history)
- **Machine-readable** (JSON API + CSV export)

---

### V. Grace Periods and Edge Cases

#### The 7-Day Grace Period (Obsidian Only)

```python
GRACE_PERIOD_DAYS = 7

# Companies get 7 consecutive days to fix failing metrics before revocation
# This prevents revocation due to:
# - Temporary infrastructure migrations
# - Cloud provider outages affecting telemetry
# - Scheduled maintenance windows

# However, the grace period is PUBLIC:
if failure_streak(company_id) > 0:
    publish_warning(
        company_id=company_id,
        message=f"Obsidian metrics failing for {failure_streak_days} days. Revocation in {7 - failure_streak_days} days if not resolved."
    )
```

#### Telemetry Outages

```python
def handle_telemetry_outage(company_id):
    """
    If telemetry stops reporting entirely (not just failing metrics),
    assume technical issue rather than performance failure.
    """
    
    if telemetry_silence_duration(company_id) > 6_hours:
        freeze_certification_status(company_id)
        publish_alert(
            company_id=company_id,
            message="Telemetry connection lost. Certification status frozen pending reconnection."
        )
        
        # If silence exceeds 72 hours, assume abandonment
        if telemetry_silence_duration(company_id) > 72_hours:
            revoke_to_silver(
                company_id=company_id,
                reason="Telemetry connection abandoned"
            )
```

#### Model Classification Disputes

```python
def handle_model_classification_appeal(company_id, task_id):
    """
    If a company believes a task was misclassified as 'routine' when it requires
    frontier reasoning, they can submit documentation for human review.
    
    This is the ONLY appeal mechanism in the system.
    """
    
    appeal = {
        "company_id": company_id,
        "task_id": task_id,
        "claimed_complexity": "frontier",
        "system_classification": "routine",
        "justification": company_submitted_documentation,
        "review_status": "pending"
    }
    
    # 3 independent technical reviewers evaluate
    # If 2/3 agree the task requires frontier models, it's reclassified
    # This does NOT grant immediate Obsidian restoration - company must meet
    # thresholds for full 30-day period after reclassification
```

---

### VI. Implementation Requirements

For this specification to function, the CMS Dashboard must:

1. **Ingest telemetry from standard cloud monitoring APIs:**
   - AWS CloudWatch
   - Azure Monitor
   - Google Cloud Operations
   - Prometheus (for on-prem/hybrid)

2. **Run certification calculations every 15 minutes** for all Obsidian-tier companies

3. **Maintain historical metric data** for minimum 365 days

4. **Publish all certification status changes** to public API within 60 seconds of calculation

5. **Provide public query endpoints:**
   - `/api/v1/company/{id}/status` (current tier)
   - `/api/v1/company/{id}/metrics` (live Obsidian metrics)
   - `/api/v1/company/{id}/history` (certification timeline)
   - `/api/v1/registry` (all certified companies)

6. **Generate anomaly detection reports** flagging suspected gaming behavior

---

### VII. Version Control and Amendment Process

**This specification is version-controlled and publicly visible.**

Changes to thresholds, grace periods, or calculation logic require:
1. Public RFC (Request for Comments) published 90 days before implementation
2. Comment period for stakeholder feedback
3. Working group vote (requires 2/3 majority)
4. 30-day notice before deployment

**Rationale:** Companies are building infrastructure to meet these standards. Sudden threshold changes would be unfair. Transparency and advance notice are mandatory.

---

### VIII. The Code is the Contract

This specification defines the complete certification logic. There are no hidden criteria, no subjective assessments (except Bronze document review and model classification appeals), no backroom decisions.

If the code calculates your status as Obsidian, you are Obsidian.
If the code revokes you to Silver, you are Silver.

**The metrics don't lie. The system doesn't negotiate.**

Fix your infrastructure or accept your tier.

---

*Carbon Mirror Standard Auto-Certification Logic Specification v1.0*  
*Published: [Date]*  
*Maintained by: CMS Working Group*  
*Source Code: github.com/carbon-mirror-standard/certification-engine*  
*License: Apache 2.0 (Open Source)*
