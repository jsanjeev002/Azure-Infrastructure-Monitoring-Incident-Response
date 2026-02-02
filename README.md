# Azure Monitoring, Alerts & Incident Response Lab

## Architecture Overview
azure-monitoring-incident-response/
│
├── README.md
│
├── architecture/
│ └── architecture-diagram.png
│
├── kql/
│ ├── cpu-spike-analysis.kql
│ ├── disk-exhaustion-analysis.kql
│ ├── vm-restart-investigation.kql
│ └── failed-login-analysis.kql
│
├── alerts/
│ ├── cpu-metric-alert.md
│ ├── disk-log-alert.md
│ ├── vm-restart-alert.md
│ └── failed-login-alert.md
│
├── incidents/
│ ├── incident-cpu.md
│ ├── incident-disk.md
│ ├── incident-vm-restart.md
│ └── incident-failed-login.md
│
├── evidence/
│ ├── alerts-fired/
│ ├── log-analytics/
│ └── activity-logs/
│
└── scripts/
├── cpu-stress.sh
└── disk-fill.sh


### Components
- Ubuntu Virtual Machine (failure target)
- Azure Monitor
- Log Analytics Workspace
- Azure Activity Logs
- Network Security Group diagnostics
- Metric-based and Log-based alerts

### Data Flow
```
Ubuntu VM
 ├─ Performance Metrics ───────────► Azure Monitor
 ├─ Syslog / VM Logs ──────────────► Log Analytics Workspace
 ├─ NSG Flow & Security Logs ──────► Log Analytics Workspace
 └─ Activity Logs (Control Plane) ─► Log Analytics Workspace
```

All investigations are performed using **KQL**.

---

## Prerequisites

You must already have:
- Active Azure subscription
- Azure Portal access
- Basic Linux command-line skills
- One Ubuntu VM

If these are missing, this project cannot work.

---

## Infrastructure Setup

### Resource Group
- **Name:** `rg-monitoring-lab`

### Virtual Machine
- **OS:** Ubuntu
- **Size:** Standard_B1s
- **Auth:** SSH key only
- **Disk:** Standard SSD
- **Enabled:**
  - Boot diagnostics
  - System-assigned managed identity

This VM is intentionally treated as a **victim system**.

---

## Log Analytics Workspace

- **Name:** `law-monitoring-lab`
- **Region:** Same as VM
- **Purpose:** Central source of truth for logs and metrics

Verified data tables:
- `Heartbeat`
- `Perf`
- `Syslog`
- `AzureActivity`

If these tables are empty, monitoring is broken.

---

## Diagnostic Logs

Diagnostics are enabled and sent to Log Analytics for:
- Virtual Machine
- Network Security Group
- Azure Activity Logs

This enables:
- VM lifecycle tracking
- Security investigations
- Configuration change auditing

---

## Alerting Strategy (Signal-Driven)

### High CPU Usage — Metric Alert
- **Signal:** Percentage CPU
- **Threshold:** > 80% for 5 minutes
- **Severity:** Medium

**Why metrics?**  
CPU utilization is a continuous numerical signal.

---

### Disk Space Exhaustion — Log Alert
- **Source:** Log Analytics (`LogicalDisk`)
- **Condition:** Free space < 20%
- **Severity:** High

**Why logs?**  
Disk incidents require mount-point and timing context, not just a value.

---

### VM Shutdown / Restart — Activity Log Alert
- **Source:** Azure Activity Logs
- **Operations:** Deallocate / Restart
- **Severity:** High

Tracks **who**, **when**, and **from where**.

---

### Failed Login Attempts — Security Log Alert
- **Source:** Syslog / Security logs
- **Condition:** Repeated authentication failures
- **Severity:** High

Used to detect brute-force or misconfiguration events.

---

## Incident Simulation

### High CPU
```bash
sudo apt install stress -y
stress --cpu 2 --timeout 300
```

---

### Disk Exhaustion
```bash
sudo fallocate -l 5G /tmp/bigfile
```

---

### VM Restart
- Restart initiated from Azure Portal

---

### Failed Login Attempts
- Multiple invalid SSH login attempts

Each simulation is verified by alert triggers and log data.

---

## Incident Investigation (KQL)

All investigations answer:
- **Who** triggered the event
- **When** it occurred
- **Why** it happened
- **What** was affected

Sample queries are stored in the `/kql` directory.

---

## Sample Incident Report

**Incident:** Disk Space Exhaustion  
**Alert Triggered:** Free space < 20%  
**Impact:** Application logs stopped writing  
**Root Cause:** Unmanaged log growth in `/var/log`  
**Resolution:** Removed old logs  
**Prevention:** Log rotation + earlier alert threshold  

This mirrors real on-call incident handling.

---

## Evidence

This repository contains:
- Alert screenshots
- Log Analytics query results
- Activity Log entries
- Incident reports

No screenshots = no proof. Evidence is mandatory.

---

## Key Takeaway

Alerts are signals.  
Logs are evidence.  
Incident response is about **answers**, not dashboards.
