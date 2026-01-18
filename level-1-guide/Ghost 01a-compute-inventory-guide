# Carbon Mirror  
## Ghost Compute Inventory Guide  
### Identifying and Eliminating Idle Infrastructure

---

## 1. Purpose

The **Ghost Compute Inventory** is designed to eliminate *“Vampire Power”* in digital architecture.  
It forces teams to identify instances, servers, or clusters that are consuming **electricity, water, and budget** while providing **no operational value**.

---

## 2. The Threshold of Waste

For the purposes of the **Carbon Mirror Standard**, an asset is classified as **Ghost Compute** if it meets **all** of the following criteria:

- **Utilization**  
  Average CPU and/or RAM utilization is **less than 10%** over a sustained **30-day period**.

- **Activity**  
  The asset has **no active network traffic or I/O requests** for more than **7 consecutive days**.

- **Redundancy**  
  The asset is a *forgotten* development, staging, or sandbox environment that is no longer tied to an active project.

---

## 3. The Audit Process

To satisfy **Level 1** requirements, teams must perform the following steps:

### 3.1 The Scan
Use cloud monitoring tools (e.g., CloudWatch, Azure Monitor, Prometheus) to pull **30-day utilization reports** for all active instances.

### 3.2 The USD Conversion
For every instance identified as **Ghost Compute**, calculate the **monthly billing cost**.  
This step intentionally converts an engineering oversight into a **financial liability**.

### 3.3 The Termination / Hibernation Plan
Document whether the asset will be:
- **Terminated** (deleted permanently), or
- **Hibernated** (paused to eliminate energy draw)

---

## 4. Inventory Log Template

| Asset Name / ID | Physical Location | Avg. Util % | Monthly USD Waste | Action Taken |
|-----------------|-------------------|-------------|-------------------|--------------|
| Staging-DB-02   | US-East-1         | 2.4%        | $440.00           | Terminated   |

---

## 5. Verification for Level 1 (Bronze Reflection)

To pass the **Bronze Reflection**, teams must:

- Identify **at least three (3)** specific Ghost Compute assets  
- Provide supporting evidence for each asset, such as:
  - A screenshot of the utilization graph, or
  - A log export proving the idle state

This evidence must clearly demonstrate that each asset met the Ghost Compute threshold criteria.
