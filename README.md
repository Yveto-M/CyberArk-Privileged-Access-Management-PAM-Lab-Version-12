# CyberArk-Privileged-Access-Management-PAM-Lab-Version-12
This project demonstrates a full end-to-end deployment of CyberArk PAM version 12 in a virtualized  environment.  The goal was to simulate a real enterprise privileged-access ecosystem—from secure vaulting of credentials to session isolation and disaster recovery—using best practices aligned with **Zero Trust** and **least privilege** principles.



## 🧩 Lab Architecture

**Core Components**
| Component | Function |
|------------|-----------|
| **Primary Vault (PrivateArk)** | Central secure repository for privileged credentials and encryption keys. |
| **PVWA (Password Vault Web Access)** | Web console for administrators and auditors to request, check out, and launch privileged sessions. |
| **CPM (Central Policy Manager)** | Automates password rotation, verification, and reconciliation for onboarded privileged accounts. |
| **PSM (Privileged Session Manager)** | Provides isolated RDP and SSH sessions without exposing credentials; all activity can be monitored and recorded. |
| **Active Directory Domain Controller** | Manages user authentication, domain joins, and Group Policy integration for CyberArk components. |
| **DR Vault (Disaster Recovery)** | Passive standby vault that replicates data from the primary vault for business continuity. |

**Network Layout**
10.1.1.1 → Primary Vault
10.1.1.2 → PVWA / CPM Server
10.1.1.3 → PSM Server
10.1.1.4 → Active Directory Domain Controller
10.1.1.5 → DR Vault


All machines were hosted in a virtual lab (VMware Workstation) with isolated internal networking for security testing.

---

## ⚙️ Implementation Highlights

- **Vault & DR Vault Configuration**
  - Installed PrivateArk Server 12.x on separate VMs.
  - Configured PADR (PrivateArk Disaster Recovery) replication with encrypted channels and heartbeat validation.
  - Verified replication integrity via Server Central Administration logs.

- **PVWA + CPM Integration**
  - Deployed IIS-based PVWA portal and linked to the primary vault.
  - Configured Central Policy Manager to automatically rotate and reconcile privileged local-admin passwords.

- **PSM Deployment**
  - Installed Remote Desktop Session Host role.
  - Registered PSM with PVWA; validated connectivity and session management status.
  - Implemented secure RDP session launching (via `PSM-RDP` connection component) to prevent credential exposure.

- **Active Directory Integration**
  - Joined PSM and management servers to the AD domain.
  - Created OU structure and Group Policies for hardened RDP access.
  - Enabled auditing and event forwarding to Splunk for monitoring.

- **Security Hardening**
  - Applied CyberArk baseline hardening scripts.
  - Enforced TLS 1.2, disabled LM/NTLMv1, and limited local admin privileges.
  - Enabled Vault audit logging and dual-control workflow for privileged account access.

---

## 🔍 Key Learnings

- Deep understanding of **Privileged Access Lifecycle Management**  
  (Vault → CPM → PSM → Audit → DR Vault).
- Hands-on configuration of **PADR replication**, **connection components**, and **policy-driven password rotation**.
- Practical experience with **Windows Server roles** (IIS, RDS, AD DS) and **infrastructure segmentation**.
- Exposure to **incident recovery** and **business continuity planning** using DR Vault activation.
- Reinforcement of **Zero Trust** and **identity-centric security** principles through controlled session brokering.

---

## 🖼️ Screenshots

| Description | Screenshot |
|--------------|-------------|
| Primary Vault and DR Vault Replication | ![Vault Replication](path/to/vaultreplication.png) |
| PVWA Accounts View | ![PVWA Accounts](path/to/pvwaaccounts.png) |
| PSM Session Management Status | ![PSM Status](path/to/psmstatus.png) |
| CPM Password Verification Logs | ![CPM Logs](path/to/cpmlogs.png) |

---

## 🧰 Tools & Technologies

- **CyberArk PAM v12** (Vault, PVWA, CPM, PSM)
- **Windows Server 2022**
- **Active Directory Domain Services**
- **VirtualBox Lab Environment**
- **PowerShell / CLI**
- **Splunk Enterprise (Logging & Monitoring)**

---

## 👤 About the Author

**Yveto Meus** — Cybersecurity Professional | IAM & PAM Engineer  
- Master’s in Cybersecurity  
- CompTIA Security+ | CyberArk Defender and Sentry Certified  
- Focus: Privileged Access Management (PAM), Identity Governance & Cloud Security  

🔗 [LinkedIn](https://www.linkedin.com/in/yvetomeus) | 💻 [GitHub](https://github.com/Yveto-M)

---

## 🚀 Next Steps

- Add **session recording and live monitoring** to PSM.
- Automate **Vault health checks and PADR verification** with PowerShell.
- Integrate **SIEM correlation** for privileged-session alerts.
- Expand the lab to include **CyberArk Credential Provider** and **Cloud Entitlements Manager**.

---
