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

---

### 🔐 Primary Vault Configuration
The **Primary Vault** is the foundation of the CyberArk PAM ecosystem. It securely stores all privileged credentials, encryption keys, and audit logs in an encrypted database.

**Key Steps**
1. Installed **CyberArk PrivateArk Server 12.x** on a dedicated Windows Server VM.  
2. Configured the Vault to operate in standalone mode with strict firewall rules.  
3. Applied CyberArk hardening baselines (disabled unused ports, enforced TLS 1.2).  
4. Created safes for PVWA, CPM, and PSM integration.  
5. Verified Vault startup and license status in **Server Central Administration**.

📸 **Screenshots**
| Description | Image |
|--------------|-------|
 |Vault Built in User
 | <img width="767" height="547" alt="built-in-user-config" src="https://github.com/user-attachments/assets/7e8bfa5e-1b72-4ec7-8a39-fd67c352da85" />|

 | Vault Hardening
 | <img width="562" height="441" alt="cyber-setup-hardening" src="https://github.com/user-attachments/assets/b608421d-e66c-4ea8-b4f8-3c399b3230c9" />|

 |Safe Creation 
 |<img width="1018" height="250" alt="vault-setup-full-complete" src="https://github.com/user-attachments/assets/b72ab5fa-13a2-47b7-9829-f90668da1bb5" />|

 |Vault Service Startup
 |<img width="1023" height="344" alt="cyberArk-server-up" src="https://github.com/user-attachments/assets/c9d285d8-f71f-40b5-bdd4-d8cbb67b1ee0" />|

---

### 🌐 Password Vault Web Access (PVWA) Configuration
Deployed the **PVWA** portal, providing a secure web interface for administrators to access credentials and launch privileged sessions.

**Key Steps**
1. Installed **IIS** and **PVWA 12.x** on a Windows Server.  
2. Linked PVWA to the **Primary Vault** using CyberArk Application Identity credentials.  
3. Verified authentication and certificate trust between PVWA and the Vault.  
4. Configured authentication policies and administrator roles.  
5. Confirmed connectivity under **Administration → Components → PSM Servers**.

📸 **Screenshots**
| Description | Image |
|--------------|-------|
| PVWA Login Page | ![PVWA Login](path/to/pvwa_login.png) |
| PVWA Configuration File | ![PVWA Config](path/to/pvwa_config.png) |
| Component Status – Connected | ![PVWA Components](path/to/pvwa_components.png) |

---

### 🔄 Central Policy Manager (CPM) Configuration
Implemented the **Central Policy Manager** (CPM) to automate password rotation, verification, and reconciliation of privileged accounts.

**Key Steps**
1. Installed CPM on a separate Windows Server linked to the Primary Vault.  
2. Configured CPM communication parameters and created CPM safes.  
3. Registered CPM in PVWA for policy automation.  
4. Tested password change and verification jobs successfully.  
5. Validated CPM logs for rotation success.

📸 **Screenshots**
| Description | Image |
|--------------|-------|
| CPM Service Running | ![CPM Service](path/to/cpm_service.png) |
| Policy Configuration in PVWA | ![CPM Policy](path/to/cpm_policy.png) |
| Password Rotation Log | ![Rotation Log](path/to/rotation_log.png) |

---

### 🧩 Active Directory Domain Integration
Integrated **Active Directory (AD)** for centralized authentication and group policy enforcement across CyberArk servers.

**Key Steps**
1. Installed and configured **Active Directory Domain Services (AD DS)** on Windows Server.  
2. Created Organizational Units (OUs) for CyberArk components.  
3. Joined PVWA, CPM, and PSM servers to the AD domain.  
4. Applied Group Policy Objects (GPOs) for RDP restrictions and password complexity.  
5. Enabled auditing and event forwarding for security monitoring.

📸 **Screenshots**
| Description | Image |
|--------------|-------|
| AD OU Structure | ![AD OU](path/to/ad_ou.png) |
| Group Policy Settings | ![GPO](path/to/gpo.png) |
| Event Forwarding | ![Event Forwarding](path/to/event_forwarding.png) |

---

### 🖥️ Privileged Session Manager (PSM) Configuration
Deployed and joined **PSM** to the AD domain to enable isolated, credential-less RDP sessions for administrators.

**Key Steps**
1. Installed **Remote Desktop Session Host (RDS)** role on the PSM server.  
2. Registered PSM with PVWA using **PSMAdminConnect** credentials.  
3. Confirmed **Session Management status = Connected** in PVWA.  
4. Deployed the **PSM-RDP Connection Component** to enable secure RDP launch via PVWA.  
5. Validated successful RDP sessions without exposing credentials.

📸 **Screenshots**
| Description | Image |
|--------------|-------|
| PSM Service Running | ![PSM Service](path/to/psm_service.png) |
| PSM Registered in PVWA | ![PSM Registered](path/to/psm_registered.png) |
| RDP Session via Connect Button | ![PSM RDP](path/to/psm_rdp.png) |

---

### 🪞 Disaster Recovery (DR) Vault Configuration
Configured a **DR Vault** to provide redundancy and business continuity through real-time replication of the Primary Vault.

**Key Steps**
1. Installed **CyberArk DR Vault 12.x** on a separate Windows Server VM.  
2. Copied configuration files (`dbparm.ini`, license, and Vault keys) from the Primary Vault.  
3. Configured **PADR replication** between the two Vaults using dedicated replication users.  
4. Verified connectivity and encryption keys in `padr.ini`.  
5. Confirmed successful replication in **Server Central Administration** logs.

📸 **Screenshots**
| Description | Image |
|--------------|-------|
| DR Vault Setup Wizard | ![DR Vault Setup](path/to/dr_vault_setup.png) |
| PADR Configuration Utility | ![PADR Config](path/to/padr_config.png) |
| Replication Sync Log | ![DR Replication](path/to/dr_replication.png) |


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
