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
> 📸 <img width="750" height="550" alt="built-in-user-config" src="https://github.com/user-attachments/assets/7e8bfa5e-1b72-4ec7-8a39-fd67c352da85" />

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
> 📸 <img width="750" height="500" alt="cyber-setup-hardening" src="https://github.com/user-attachments/assets/b608421d-e66c-4ea8-b4f8-3c399b3230c9" />

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
> 📸 <img width="750" height="500" alt="cyberArk-server-up" src="https://github.com/user-attachments/assets/c9d285d8-f71f-40b5-bdd4-d8cbb67b1ee0" />

For anyone new to CyberArk, seeing these safes appear is confirmation that your Vault is alive, healthy, and successfully encrypted.

---

### Step 4: Validating Vault Services

Before moving on, I validated the Vault’s services through **Server Central Administration (SCA)**. This step ensures all necessary services—Vault, PrivateArk Server, and communication listeners—are active and using the correct encryption algorithms (AES-256, RSA-2048, SHA-512).

> **Figure 4.** Vault services running and communication logs active
> 📸 <img width="750" height="500" alt="vault-setup-full-complete" src="https://github.com/user-attachments/assets/b72ab5fa-13a2-47b7-9829-f90668da1bb5" />


The SCA log output showed successful startup, database mount confirmation, and secure socket binding—all key indicators of a stable Vault installation.

---

### Step 5: Network Isolation and IPv4 Configuration

With the Vault now operational, I tightened its network configuration. Following CyberArk best practices, I disabled **IPv6** and unnecessary discovery protocols to minimize exposure.

This ensures the Vault operates as a **“bubble system”**—reachable only from explicitly defined servers like PVWA or CPM, and invisible to everything else.

> **Figure 5.** IPv4-only configuration and host isolation
> 📸 <img width="300" height="400" alt="Ip-config-v4" src="https://github.com/user-attachments/assets/5ac8fb6e-804c-4376-8c96-82dcc10d6406" />


Restricting the Vault to IPv4 simplifies firewall rule management and aligns with real-world enterprise hardening standards.

---

### Key Takeaways

💡 **Security-first design:** Implemented CyberArk’s built-in hardening and network isolation principles for maximum protection.
💡 **Clarity of architecture:** Demonstrated understanding of how the Vault acts as the root of trust for all downstream PAM components.
💡 **Operational validation:** Verified encryption, core safes, and service health using administrative tools.
💡 **Production mindset:** Built and validated this environment with real-world deployment rigor.

---


## 🌐 Phase 2 — Deploying CyberArk Password Vault Web Access (PVWA)

### Overview

After standing up the hardened Production Vault, the next step was deploying **Password Vault Web Access (PVWA)** — the web-based interface that users, administrators, and auditors rely on for secure, policy-driven access to privileged accounts.

The PVWA serves as the **front door** to the CyberArk environment. It connects securely to the Vault over an encrypted channel, allowing authorized users to view and manage privileged credentials without ever exposing them in plain text.

This phase started with building a dedicated Windows Server VM, installing IIS and required web components, then configuring PVWA to integrate seamlessly with the Vault.

---

### Step 1: Preparing the Server Environment

I provisioned a new **Windows Server 2012 R2 Datacenter** virtual machine using **VMware Workstation Pro 17**. The PVWA server was given a static IP and configured to sit within the same internal network as the Vault (to allow controlled communication over TCP port 1858).

> **Figure 1.** Creating a new Windows Server 2012 R2 VM for PVWA
> 📸 <img width="450" height="450" alt="VM-For-PVWA" src="https://github.com/user-attachments/assets/e47d90a2-e371-423d-ba94-c7534ddf5454" />


Once the virtual machine was created, I performed initial OS setup — product key activation, account creation, and system configuration.

> **Figure 2.** Windows Server setup and personalization during VM creation
> 📸 <img width="400" height="400" alt="pvwa-win12-product-code" src="https://github.com/user-attachments/assets/4766c7ac-9465-4cdf-aa52-f8a23ab8dc01" />


With the base OS ready, I installed all available Windows updates, assigned the appropriate hostname, and verified network connectivity to the Vault server.

---

### Step 2: Installing IIS and Web Components

Before installing PVWA, I had to ensure **Internet Information Services (IIS)** and its dependencies were present. PVWA runs as a secure web application hosted under IIS, which handles HTTPS requests and authentication.

Through **Server Manager → Add Roles and Features**, I installed the required features:

* .NET Framework 3.5 & 4.5
* ASP.NET 4.5
* Web Server (IIS) → Common HTTP Features (Default Document, Static Content, Directory Browsing, HTTP Errors)

> **Figure 3.** Installing IIS and related roles
> 📸 <img width="550" height="550" alt="pre-req-install-roles-for-pvwa" src="https://github.com/user-attachments/assets/2625a1e2-4934-4e57-bcb5-d9c49be16800" />

After installation, I opened the IIS Manager to confirm the **Default Web Site** was created and bound to both HTTP (port 80) and HTTPS (port 443).

> **Figure 4.** IIS web site bindings showing HTTP and HTTPS configuration
> 📸 <img width="600" height="500" alt="Binding-SSL" src="https://github.com/user-attachments/assets/4ec865b3-8680-4918-af87-db59fbc5e5e9" />


---

### Step 3: Running the PVWA Installation Wizard

With prerequisites in place, I launched the **PVWA v12 InstallShield Wizard**. This installer guided me through the configuration — defining the web application name, authentication type, and Vault connection details.

> **Figure 5.** PVWA installation wizard welcome screen
> 📸 <img width="539" height="412" alt="pvwa-initial-install-step-1" src="https://github.com/user-attachments/assets/f2486aca-29af-4582-96b6-aac2be6ced63" />


During setup, I selected **CyberArk Authentication** as the default type to ensure native Vault authentication integration.

> **Figure 6.** Selecting authentication type during PVWA configuration
> 📸 <img width="525" height="403" alt="PVWA-ini-install-2" src="https://github.com/user-attachments/assets/0290efe7-2a86-446f-8400-71f1a8a0dc28" />


Next, I provided the **Vault connection details**, specifying the Vault’s secure endpoint (IP and port 1858) and defining the PVWA base URL.

> **Figure 7.** Defining Vault connection and PVWA URL
> 📸 <img width="530" height="408" alt="pvwa-ini-install-3-IP-change" src="https://github.com/user-attachments/assets/c9aecc18-1512-4747-ae6c-895f478c9378" />


Then, the **Vault Administrator credentials** were entered — this allows PVWA to register itself with the Vault and complete its server-side configuration securely.

> **Figure 8.** Providing Vault Administrator credentials
> 📸 <img width="522" height="406" alt="pvwa-ini-install-4-adminVault-credentials" src="https://github.com/user-attachments/assets/11934646-9dac-4175-822e-7261b03e5595" />


When the installation completed successfully, PVWA confirmed setup and deployment on version 12.0.0.

> **Figure 9.** PVWA installation completed
> 📸 <img width="540" height="406" alt="pvwa-ini-install-5-complete" src="https://github.com/user-attachments/assets/af480424-5c3e-42f8-b51b-8620979893ae" />

---

### Step 4: Enabling TLS 1.2 and Running PVWA Hardening

After installation, I executed the **PVWA Hardening PowerShell scripts** provided by CyberArk. These scripts:

* Disable weak SSL/TLS versions
* Enforce **TLS 1.2** as the minimum encryption protocol
* Configure the IIS SSL bindings and redirect HTTP → HTTPS
* Harden system services for PVWA

> **Figure 10.** PVWA prerequisites and hardening scripts executing
> 📸 <img width="700" height="323" alt="IIS-server-Installed" src="https://github.com/user-attachments/assets/a4fe7496-1c26-4d99-89ac-f11eb6fe568a" />

> 📸 <img width="700" height="398" alt="PVWA-hardening" src="https://github.com/user-attachments/assets/4af06788-040f-4b00-9b2b-48d09962423c" />


The PowerShell output confirmed TLS 1.2 enforcement and successful completion of the PVWA hardening process.

---

### Step 5: Validating the Web Interface

After restarting IIS and confirming all CyberArk services were running, I tested access by browsing to the PVWA URL over HTTPS.

> **Figure 11.** Accessing the PVWA web portal over HTTPS
> 📸 <img width="600" height="550" alt="PVWA-PostC-web-login" src="https://github.com/user-attachments/assets/fdcc7cf8-d18a-4b7a-ba3a-e90eb0701b2e" />


The login page loaded successfully, confirming PVWA connectivity with the Vault and the correct SSL configuration.

---

### Key Takeaways

💡 **End-to-end setup:** Built and configured a dedicated Windows Server VM, installed IIS, and successfully deployed PVWA v12.
💡 **Security alignment:** Enforced TLS 1.2, HTTPS-only access, and hardening scripts to align with CyberArk security standards.
💡 **Integration mastery:** Connected PVWA to the hardened Production Vault using secure administrative authentication.
💡 **Real-world readiness:** The web console is fully functional — ready for CPM, PSM, and DR Vault integration.


---

## ⚙️ Phase 3 — Deploying and Integrating the CyberArk Central Policy Manager (CPM)

### Overview

The **Central Policy Manager (CPM)** is the engine that automates password rotation, verification, and reconciliation across all managed systems. While enterprise deployments host CPM on a dedicated server, in this lab I co-hosted CPM with **PVWA** to streamline resource use and focus on integration behavior.

This phase covers the full lifecycle: installing CPM, securing communication with the Vault, creating self-signed certificates, resolving SSL validation errors, completing hardening, and verifying automation through a functional “VetoLabs” safe.

---

### Step 1 — Launching the CPM Installer

After preparing the PVWA server, I extracted the **CPM v12.0 installer** and ran the setup executable as Administrator.

> **Figure 1.** Running the CPM installer with elevated privileges
> 📸 <img width="789" height="345" alt="cpm-ini-install-run-as-admin" src="https://github.com/user-attachments/assets/3949efcd-0f2d-4a77-835b-92e8a7b262ae" />

The installer confirmed required prerequisites (.NET, IIS components, Vault connectivity) and began configuration.

---

### Step 2 — Vault Integration Credentials

The setup prompted for the **Vault Administrator credentials** so CPM could register itself with the Vault and receive its encryption keys.

> **Figure 2.** Entering Vault Administrator credentials during CPM setup
> 📸 <img width="538" height="408" alt="cpm-set-up-admin-creden-1" src="https://github.com/user-attachments/assets/31a4df14-978c-4ea7-9810-cb8e8fbc649e" />

This registration step binds CPM’s internal “PasswordManagerUser” to the Vault, enabling policy-based password management.

---

### Step 3 — Creating and Installing SSL Certificates

To establish mutual trust between components, I created a **self-signed certificate** for the CPM server and imported the PVWA SSL certificate into its Trusted Root store.

> **Figure 3.** Generating a self-signed certificate for CPM
> 📸 <img width="683" height="331" alt="creating-ssCERT-4" src="https://github.com/user-attachments/assets/74e412a7-59c1-4049-8f3a-1d677bc19dd9" />

> **Figure 4.** Exporting the certificate for reuse
> 📸 <img width="551" height="259" alt="EXPORT-CERT-5" src="https://github.com/user-attachments/assets/b413a593-0ebd-417f-935d-5b653f5b1dd9" />

> **Figure 5.** Importing PVWA SSL certificate to local machine
> 📸 <img width="804" height="192" alt="SSL-cert-cpm" src="https://github.com/user-attachments/assets/0e05b99c-e4f0-48b2-9889-ade4a7fa9eb5" />

> **Figure 6.** Placing the certificate into Trusted Root store
> 📸 <img width="967" height="239" alt="Importing-SSL-Cert-to-trusted root" src="https://github.com/user-attachments/assets/3a8d6c01-78e2-4138-848a-29de067f2d4d" />


Proper certificate placement ensures encrypted Vault-to-CPM communication and prevents the common “Untrusted Root” SSL chain error.

---

### Step 4 — Resolving SSL and Service Issues

During first startup, the **CPM Scanner service** threw an SSL validation error (`UntrustedRoot`). I traced it to an incomplete certificate chain using the `CACPMScanner.log`.

> **Figure 7.** SSL validation error detected in Scanner log
> 📸 <img width="776" height="336" alt="Tshoot-scanner-SSL-ERROR" src="https://github.com/user-attachments/assets/972f1116-005f-4b7e-9be3-c6db31cbaed7" />


After importing the certificate into the correct store and restarting the service, the error cleared and all CPM services reached the **Running** state.

> **Figure 8.** All CPM services running under PasswordManagerUser
> 📸 <img width="640" height="142" alt="cpm-scanning-comp-running-9" src="https://github.com/user-attachments/assets/7418d9da-89c8-438f-996c-154ffb4864bd" />


This fix highlights the importance of validating certificate trust paths in CyberArk’s distributed architecture.

---

### Step 5 — Hardening and Vault File Validation

Next, I executed the **CPM Hardening PowerShell script**, which enforces local policy lockdowns, restricts folder permissions, and disables non-essential services.

> **Figure 9.** Hardening script execution confirmation
> 📸 <img width="827" height="121" alt="proof-fo-harnening-success" src="https://github.com/user-attachments/assets/cccdfb08-e57b-4a05-86a4-07661b3f6d6e" />


I also replaced the local `Vault.ini` with the updated version from the Vault server to ensure proper API key alignment.

> **Figure 10.** Replacing Vault INI file to sync API keys
> 📸 <img width="803" height="386" alt="replace-vault-api-keys" src="https://github.com/user-attachments/assets/13a8452f-3fb5-4389-a39a-f3714fffc6b3" />


With this, the CPM instance became fully authorized to interact with the Vault.

---

### Step 6 — Certificate Verification and Trusted Communication

Finally, I verified that CPM’s self-signed certificate was live in production and bound correctly to the HTTPS listener.

> **Figure 11.** Certificate successfully bound to CPM listener
> 📸 <img width="692" height="291" alt="SS-cert-inproduction" src="https://github.com/user-attachments/assets/ea623edd-1aff-4b29-a42d-f5a0bf803397" />

> **Figure 12.** Confirming SSL binding configuration
> 📸 <img width="588" height="470" alt="SSL-on-CPM-Installation" src="https://github.com/user-attachments/assets/e51de4ca-11cb-4a5f-b197-56188f0ec790" />


This completed the secure channel setup, ensuring every password operation between CPM and Vault travels through TLS 1.2 encryption.

---

### Step 7 — Safe Creation and Policy Assignment

Using PVWA, I created a new safe named **VetoLabs** and assigned **PassManagerPro** as its responsible CPM.

> **Figure 13.** Creating the “VetoLabs” safe and assigning to CPM
> 📸 <img width="716" height="364" alt="Adding-safe-toCPM-10" src="https://github.com/user-attachments/assets/d02aff2d-5dfc-4a4d-b0a2-41669cb14be7" />


Within minutes, CPM detected the new safe, initiated account verification, and scheduled rotations per policy.

---

### Step 8 — Account Onboarding and Verification

To test CPM automation, I onboarded a local Windows Server account under the VetoLabs safe.

> **Figure 14.** Account successfully onboarded and verified by CPM
> 📸 <img width="897" height="415" alt="cpm-account-onboard-verified-11" src="https://github.com/user-attachments/assets/a20dcbc8-7ab4-4af6-ad49-69473d790eb0" />


In the Vault, I confirmed the creation of new CPM safes (cpm-old versions, PassManagerPro safe, etc.), showing that the Vault–PVWA–CPM integration was working end-to-end.

> **Figure 15.** Updated Vault safes list reflecting CPM registration
> 📸 <img width="1021" height="352" alt="cpm-complete" src="https://github.com/user-attachments/assets/7a627ed5-bcc6-4f67-9eae-0d09d4031d1e" />


---

### Key Takeaways

💡 **Automation Achieved:** CPM successfully rotates and manages credentials stored in the Vault through policy enforcement.
💡 **Hands-on Troubleshooting:** Resolved SSL and service startup issues, strengthening understanding of CyberArk infrastructure dependencies.
💡 **Security Best Practices:** Executed CPM hardening and TLS certificate management to ensure secure machine-to-machine communication.
💡 **Integration Fluency:** Demonstrated end-to-end connectivity between Vault, PVWA, and CPM within a co-hosted lab environment.

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
