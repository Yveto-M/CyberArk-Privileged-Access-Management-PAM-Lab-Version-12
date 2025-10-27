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

Perfect — I’ll revise your **Production Vault section** to:

1. Keep the same engaging blog-meets-portfolio tone.
2. Add smooth explanations *between* screenshots (so it reads like a full story without showing every click).
3. Include clearly labeled **screenshot placeholders** where you can just paste images later (with markdown paths).

Here’s your updated version 👇

---

## 🧱 Phase 1 — Building the CyberArk Production Vault

### Overview

The **CyberArk Digital Vault** is the beating heart of any PAM deployment. Every other component—PVWA, CPM, PSM, and the DR Vault—depends on it. In this phase, I focused on deploying and hardening the **Primary Vault**, ensuring it was production-ready, fully isolated, and compliant with CyberArk’s hardening guidelines.

Think of this phase as laying the foundation of a secure fortress. The steps that follow—component setup, Active Directory integration, and DR replication—all rely on the integrity and configuration of this core Vault.

---

### Step 1: Creating the Built-In Accounts (Master & Administrator)

The installation starts with defining two critical built-in accounts: **Master** and **Administrator**.

* **Master** is the break-glass recovery account—used only in emergencies or Vault restoration scenarios.
* **Administrator** manages daily Vault operations and system configuration.

These accounts enforce role separation—one of CyberArk’s core security principles. Between these two, the Vault maintains both recoverability and operational stability.

> **Figure 1.** Creating the built-in Master and Administrator accounts
> 📸 *Paste screenshot here:* `./screenshots/built-in-user-config.png`

Once these credentials were set, the Vault generated its initial encryption keys and prepared the machine for hardening. At this stage, no external components (PVWA, CPM, or PSM) can connect yet—the Vault first secures its environment.

---

### Step 2: Automatic Hardening

After account creation, CyberArk’s **Automatic Hardening** begins. This is where the Vault differentiates itself from a typical Windows server—it transforms into a locked-down security appliance.

During this process, the installer:

* Disables unnecessary Windows services and shares.
* Applies restrictive NTFS permissions.
* Configures a dedicated Windows firewall profile.
* Sets system-level policies that align with CyberArk’s *Digital Vault Security Standard.*

This is one of my favorite steps—it’s where you can literally watch the system secure itself.

> **Figure 2.** Hardening in progress — CyberArk securing the operating system
> 📸 *Paste screenshot here:* `./screenshots/cyber-setup-hardening.png`

When the hardening completes, the Vault machine is effectively isolated and production-grade secure.

---

### Step 3: Vault Initialization & Core Safes

Once installation and hardening were complete, the Vault initialized its **core safes**, the internal repositories that make up its database.

These include:

* `System` — internal configuration and maintenance data.
* `VaultInternal` — operational metadata and log indexes.
* `Notification Engine` — event-driven alerting mechanisms.

At this stage, the Vault is fully operational—ready to accept connections from other CyberArk components.

> **Figure 3.** Vault safes confirmed post-installation
> 📸 *Paste screenshot here:* `./screenshots/vault-setup-full-complete.png`

For anyone new to CyberArk, seeing these safes appear is confirmation that your Vault is alive, healthy, and successfully encrypted.

---

### Step 4: Validating Vault Services

Before moving on, I validated the Vault’s services through **Server Central Administration (SCA)**. This step ensures all necessary services—Vault, PrivateArk Server, and communication listeners—are active and using the correct encryption algorithms (AES-256, RSA-2048, SHA-512).

> **Figure 4.** Vault services running and communication logs active
> 📸 *Paste screenshot here:* `./screenshots/cyberArk-server-up.png`

The SCA log output showed successful startup, database mount confirmation, and secure socket binding—all key indicators of a stable Vault installation.

---

### Step 5: Network Isolation and IPv4 Configuration

With the Vault now operational, I tightened its network configuration. Following CyberArk best practices, I disabled **IPv6** and unnecessary discovery protocols to minimize exposure.

This ensures the Vault operates as a **“bubble system”**—reachable only from explicitly defined servers like PVWA or CPM, and invisible to everything else.

> **Figure 5.** IPv4-only configuration and host isolation
> 📸 *Paste screenshot here:* `./screenshots/Ip-config-v4.png`

Restricting the Vault to IPv4 simplifies firewall rule management and aligns with real-world enterprise hardening standards.

---

### Key Takeaways

💡 **Security-first design:** Implemented CyberArk’s built-in hardening and network isolation principles for maximum protection.
💡 **Clarity of architecture:** Demonstrated understanding of how the Vault acts as the root of trust for all downstream PAM components.
💡 **Operational validation:** Verified encryption, core safes, and service health using administrative tools.
💡 **Production mindset:** Built and validated this environment with real-world deployment rigor.

---


<img width="750" height="750" alt="built-in-user-config" src="https://github.com/user-attachments/assets/7e8bfa5e-1b72-4ec7-8a39-fd67c352da85" />**Vault Built in User** 

<img width="750" height="750" alt="cyber-setup-hardening" src="https://github.com/user-attachments/assets/b608421d-e66c-4ea8-b4f8-3c399b3230c9" />**Vault Hardening**

<img width="750" height="750" alt="cyberArk-server-up" src="https://github.com/user-attachments/assets/c9d285d8-f71f-40b5-bdd4-d8cbb67b1ee0" />**Vault Service Startup**

<img width="750" height="750" alt="vault-setup-full-complete" src="https://github.com/user-attachments/assets/b72ab5fa-13a2-47b7-9829-f90668da1bb5" />**Safe Creation**


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
