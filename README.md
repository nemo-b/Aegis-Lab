# 🛡️ Project Aegis — DNS Sinkhole & SIEM Telemetry Pipeline

**Goal:** Engineer a secure, containerized DNS resolver and establish a robust data pipeline to **Splunk SIEM** for real-time network visibility and threat detection.

## 🧭 Project Phases

This lab follows an engineering-first approach to infrastructure and monitoring:

- **Phase 1:** Containerized Deployment (Pi-hole v6 via Docker Compose)
- **Phase 2:** Persistence Engineering (Docker Volume Mapping & FTL Config)
- **Phase 3:** SIEM Integration (Splunk Universal Forwarder & File Monitoring)
- **Phase 4:** Data Visualization (App mapping & Dashboarding)
- **Phase 5:** Technical Validation (Network bypass testing & Troubleshooting)

## 🔍 Guiding Questions

- Can containerized logs be reliably persisted to a host OS for security auditing?
- How does an analyst bypass OS-level networking constraints (Port 53) to validate service health?
- Can raw DNS queries be transformed into high-fidelity security alerts within Splunk?

## 🛠️ Tools Used

- **Docker & Docker Compose:** Container orchestration and Infrastructure-as-Code (IaC).
- **Pi-hole v6:** Network-wide DNS sinkhole and FTL engine.
- **Splunk Enterprise:** SIEM platform for data ingestion and visualization.
- **YAML/Bash:** Configuration management and CLI-based troubleshooting.

## 📄 Documentation & Engineering Logs

*Phase 5*: Resolved a critical service failure by identifying Port 53 contention between the Windows 'Shared Access' service and the Docker environment, restoring network-wide filtering through IaC (Infrastructure as Code) adjustments.

- **[Project-Aegis-Notes]()** _This document contains task checklists, YAML configurations, and reflections on overcoming hybrid-OS networking challenges._

## 📸 Technical Proof

**Splunk Telemetry Validation:**
<p align="center">
  <img src="images/Getting%20Splunk%20Logs%20from%20Pi-Hole.png" alt="Getting Splunk Logs from Pi-Hole" width="500">
</p>

_Confirmation of successful data ingestion showing raw DNS query logs in the Splunk main index._

**Security Analytics Dashboard:**
<p align="center">
  <img src="images/Splunk%20Logs%20from%20the%20App%20Veiw.png" alt="Splunk Logs from the App Veiw" width="500">
</p>

_The final SOC view: Visualizing permitted vs. blocked traffic across the virtualized network._

## 📝 Key Learning Outcomes

This project reinforced the reality of **Systems Engineering**: dealing with ambiguous failures, port contention, and version-specific API changes. It proved that the most critical skill for a SOC Analyst is not just using tools, but understanding the underlying plumbing that connects them.
