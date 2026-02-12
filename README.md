# 🛡️ End-to-End Syslog Ingestion via Azure Monitor Agent (AMA)

## 📌 Project Overview

This project demonstrates the end-to-end configuration of a **Cloud-Native SIEM (Microsoft Sentinel)** to ingest, parse, and analyze security logs from a Linux-based Syslog forwarder. I successfully bridged the gap between a local Linux environment and the Azure cloud using the modern **Azure Monitor Agent (AMA)** and **Data Collection Rules (DCR)**.

## 🎯 Aim & Objective

The primary goal was to establish a secure, scalable log ingestion pipeline. This lab showcases the transition from legacy monitoring methods to the modern Azure architecture, providing a production-ready template for centralized log forwarding and security orchestration.

---

## 📋 Prerequisites

Before deployment, the following environment requirements were met:

* **Active Azure Subscription:** Trial/Pay-As-You-Go account.
* **Log Analytics Workspace (LAW):** Configured as the central data repository.
* **Sentinel Contributor Permissions:** To enable SIEM solutions and DCRs.
* **Network Planning:** Identified local Public IP for secure SSH whitelisting.
* **Linux Fundamentals:** Familiarity with Bash, `rsyslog` configurations, and SSH.

---

## 🏗️ Architectural Journey

The data follows a specific path from the edge to the cloud:

1. **Source:** A Linux Ubuntu 24.04 VM acting as a centralized log collector.
2. **Listener:** `rsyslog` daemon configured to listen on Port **514 (UDP/TCP)**.
3. **Processor:** **Azure Monitor Agent (AMA)** installed on the VM to parse incoming data.
4. **Router:** **Data Collection Rule (DCR)** defining the logic for data filtering and routing.
5. **Destination:** **Log Analytics Workspace** integrated with **Microsoft Sentinel**.

---

## 🛠️ Implementation Details

### Step 1: Resource Provisioning & Networking

* **Compute:** Deployed an **Ubuntu 24.04 LTS** VM.
* **Security (NSG):** Implemented the **Principle of Least Privilege**.
* **SSH (Port 22):** Restricted strictly to my verified Public IP.
* **Syslog (Port 514):** Opened for UDP/TCP traffic to allow external log reception.



### Step 2: AMA & DCR Configuration

Instead of manual installation, the agent was deployed via the DCR "Associate" feature.

* **Facilities:** Captured `AUTH`, `AUTHPRIV`, `DAEMON`, `SYSLOG`, and `LOCAL0-7`.
* **Severities:** Set to `LOG_INFO` to capture all critical events while filtering out unnecessary `DEBUG` noise.

### Step 3: Log Forwarder Scripting

Executed the Microsoft-provided Python installation script to configure the internal `rsyslog` daemon to communicate with the AMA.

```bash
# Install Forwarder components
sudo wget -O Forwarder_AMA_installer.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/Syslog/Forwarder_AMA_installer.py && sudo python3 Forwarder_AMA_installer.py

# Verify the listener is active
sudo netstat -anup | grep 514

```

---

## 🚧 Challenges & Troubleshooting

* **Regional Capacity:** Encountered `NotAvailableForSubscription` in East US Zone 1.
* *Resolution:* Pivoted deployment to **Zone 2** after analyzing regional availability.


* **Dynamic Source IP:** ISP IP changed mid-lab, breaking SSH access.
* *Resolution:* Updated NSG rules with the new "What is my IP" address to restore connectivity.


* **Ingestion Latency:** Initial logs did not appear immediately in the console.
* *Resolution:* Verified `azuremonitoragent` service status on the VM and allowed for the 10-minute cloud ingestion buffer.



---

## 🧠 Key Learnings

* **Agent Evolution:** Mastered the **AMA** and understood its advantages over the legacy Log Analytics Agent.
* **Syslog Granularity:** Learned how to tune Facilities and Severities to optimize ingestion costs and signal-to-noise ratios.
* **KQL Proficiency:** Developed custom queries to validate data flow and monitor system health.

---

## 📊 Project Validation (KQL)

To verify the pipeline, I manually injected a test log on the VM and queried it in Sentinel.

**Manual Trigger:**

```bash
logger -p local0.info "SENTINEL_SUCCESS_CHECK: System is online. Samson you rock"

```

**Sentinel Query:**

```kusto
Syslog
| where SyslogMessage has "SENTINEL_SUCCESS_CHECK"
| project TimeGenerated, Computer, Facility, SeverityLevel, SyslogMessage

```

---

### 📂 Repository Structure

* **`/images`**: Screenshots of Azure Portal configurations and KQL results.
* **`/scripts`**: Any custom scripts used for log generation or testing.
* **`README.md`**: Project documentation.

---


