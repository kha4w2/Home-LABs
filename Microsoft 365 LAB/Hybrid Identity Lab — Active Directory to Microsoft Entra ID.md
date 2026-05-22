# Hybrid Identity Lab — Active Directory to Microsoft Entra ID

---

## Introduction

This lab documents the end-to-end implementation of a hybrid identity environment, bridging an on-premises Active Directory infrastructure with Microsoft Entra ID (formerly Azure AD). The environment is built entirely on VMware Workstation using isolated virtual machines, simulating a real enterprise network topology.

The lab covers domain controller deployment, organizational unit design, user and group provisioning, identity synchronization via Microsoft Entra Connect, and verification of synced objects in the cloud tenant.

---

## Objectives

- Deploy and configure a Windows Server 2019 Domain Controller in an isolated lab network
- Design and implement an Active Directory structure with OUs, users, and security groups
- Install and configure Microsoft Entra Connect on the Domain Controller
- Synchronize a targeted OU to Microsoft Entra ID using scoped sync
- Verify hybrid identity objects in the Entra ID portal

---

## Tools & Technologies

| Category | Tool / Technology |
|---|---|
| Hypervisor | VMware Workstation |
| Domain Controller OS | Windows Server 2019 Standard |
| Client OS | Windows 10 (22H2) |
| Directory Service | Active Directory Domain Services (AD DS) |
| Cloud Identity | Microsoft Entra ID (P2 License) |
| Sync Engine | Microsoft Entra Connect Sync |
| Identity Management Portal | Microsoft Entra Admin Center / Azure Portal |
| Scripting | Windows PowerShell |
| Network Design | VMware LAN Segment + Bridged Adapter |

---

## Conceptual Questions

### 1. What is the difference between a Microsoft 365 Group and a Security Group?

| Feature | Microsoft 365 Group | Security Group |
|---|---|---|
| Purpose | Collaboration (Teams, SharePoint, Exchange) | Access control and permissions |
| Has Mailbox | Yes | No |
| Can be used for app/resource permissions | Limited | Yes |
| Created in | Microsoft 365 / Entra ID | Active Directory or Entra ID |
| Synced from on-prem AD | No | Yes |
| Supports dynamic membership | Yes | Yes (Entra ID P1/P2) |

**In short:** M365 Groups are collaboration containers. Security Groups are used to grant or restrict access to resources.

---

### 2. Dynamic User, Dynamic Group, and Assigned — What's the difference?

**Assigned:** Membership is managed manually by an administrator. Users are added or removed explicitly.

**Dynamic User:** A user account whose properties (like department, job title, or location) automatically trigger group membership based on a defined rule. Example rule: `department -eq "IT"` — any user with IT as their department automatically joins the group.

**Dynamic Group:** A group whose membership is automatically maintained by Entra ID based on attribute-based rules. No manual additions are needed — the engine evaluates rules continuously and adjusts membership as attributes change.

> Requires **Entra ID P1 or P2** license for dynamic membership.

---

### 3. Authentication Methods in Microsoft Entra ID

| Method | How It Works | Best For |
|---|---|---|
| **Password Hash Sync (PHS)** | AD password hashes are synced to Entra ID. Authentication happens in the cloud. | Simplest hybrid setup, high availability |
| **Pass-Through Authentication (PTA)** | Authentication request is forwarded to on-prem AD in real time. Passwords never leave the network. | Compliance-driven orgs that cannot store hashes in cloud |
| **Federation (AD FS)** | Entra ID redirects auth to a federated identity provider (AD FS). Full SSO experience. | Complex enterprise SSO with smart cards or MFA tokens |
| **Cloud-Only Authentication** | Users exist only in Entra ID. No on-prem dependency. | Cloud-native organizations |
| **Microsoft Authenticator (MFA)** | Push notification, OTP, or passwordless sign-in via the app | MFA enforcement for all users |
| **FIDO2 Security Keys** | Hardware keys for passwordless authentication | High-security environments |
| **Windows Hello for Business** | Biometric or PIN-based authentication tied to device | Modern workplace with Hybrid or Entra joined devices |

---

## Lab Implementation

---

## Phase 1 — Network Configuration

Before deploying Active Directory, both virtual machines must be networked correctly. The Domain Controller requires two adapters: one for internet access (Bridged) and one for internal communication with the Windows 10 client (LAN Segment). The Windows 10 machine uses only the LAN Segment adapter, routing all traffic through the DC.

---

### Step 1 — Configure VMware Network Adapters

Each virtual machine is assigned the correct network adapter type inside VMware Workstation settings. The Server gets two adapters; the client gets one. This creates an isolated internal network while giving the server internet access through the physical machine.

**Figure 1 — Windows Server 2019: Bridged Adapter (Internet)**
![Figure 1](images/image1.png)

**Figure 2 — Windows Server 2019: LAN Segment Adapter (Internal Network)**
![Figure 2](images/image2.png)

---

### Step 2 — Open Server Manager and Review Initial State

After booting Windows Server 2019, Server Manager opens automatically. The initial state shows the machine in WORKGROUP with no domain, and both network adapters visible. Three configurations are required before promotion: time zone, machine name, and static IP on the LAN adapter.

**Figure 3 — Server Manager Local Server: Initial State Before Configuration**
![Figure 3](images/image3.png)

---

### Step 3 — Set Static IP on the LAN-5 Adapter

The LAN-5 (internal) adapter is configured with a static IP address so it can serve as a reliable gateway and DNS server for the Windows 10 client. The Bridged adapter is left as DHCP to receive internet from the physical network.

**Figure 4 — Opening Network Connections via Network and Sharing Center**
![Figure 4](images/image4.png)

**Figure 5 — Accessing IPv4 Properties on the LAN-5 Adapter**
![Figure 5](images/image5.png)

**Figure 6 — Static IP Configuration: 192.168.10.1 / DNS: 127.0.0.1**
![Figure 6](images/image6.png)

---

### Step 4 — Verify Server Network Configuration in Server Manager

After applying the static IP, Server Manager reflects the updated network state. The Internet adapter shows DHCP (internet access) and LAN-5 shows the assigned static IP.

**Figure 7 — Server Manager Confirming Dual Adapter Configuration**
![Figure 7](images/image7.png)

---

### Step 5 — Configure Windows 10 Static IP and Test Connectivity

The Windows 10 client is configured with a static IP in the same subnet as the server's LAN adapter, pointing to the server as both gateway and DNS. Connectivity is confirmed by pinging the DC's LAN IP.

**Figure 8 — Windows 10 Static IP Configuration Pointing to DC**
![Figure 8](images/image8.png)

**Figure 9 — Successful Ping from Windows 10 to Domain Controller (0% Packet Loss)**
![Figure 9](images/image9.png)

> ✅ Phase 1 Complete — Network is fully operational. Both machines communicate over LAN-5 and the server has internet access via the Bridged adapter.

---

## Phase 2 — Domain Controller Promotion

With networking in place, the Windows Server 2019 machine is promoted to a Domain Controller. This establishes the on-premises Active Directory forest `GBG.local`, which will later be synced to Entra ID.

---

### Step 6 — Initiate Domain Controller Promotion

The AD DS role was previously installed. The post-deployment notification in Server Manager triggers the promotion wizard.

**Figure 10 — Server Manager Post-Deployment Notification: Promote to Domain Controller**
![Figure 10](images/image10.png)

---

### Step 7 — Configure New Forest and Domain Name

The wizard is set to create a new forest. The root domain name is set to `GBG.local`, which will serve as the on-premises domain throughout the lab.

**Figure 11 — AD DS Configuration Wizard: Add a New Forest — Root Domain: GBG.local**
![Figure 11](images/image11.png)

---

### Step 8 — Set Domain Controller Capabilities and DSRM Password

Forest and domain functional levels are set to Windows Server 2016. DNS Server and Global Catalog are enabled. The DSRM recovery password is configured.

**Figure 12 — Domain Controller Options: Functional Levels, DNS, GC, and DSRM Password**
![Figure 12](images/image12.png)

---

### Step 9 — Pass Prerequisites Check and Install

All prerequisite checks pass successfully. Warnings about static IP and DNS delegation are expected in a lab environment and do not block installation. The Install button is clicked and the server reboots automatically.

**Figure 13 — Prerequisites Check Passed: All Checks Successful — Ready to Install**
![Figure 13](images/image13.png)

**Figure 14 — Server Restarting After Domain Controller Promotion**
![Figure 14](images/image14.png)

---

### Step 10 — Verify Domain Controller Is Active

After reboot, Server Manager confirms the machine is now a member of the `GBG.local` domain, no longer in WORKGROUP. Both adapters remain configured correctly.

**Figure 15 — Server Manager Confirming Domain: GBG.local — DC Promotion Successful**
![Figure 15](images/image15.png)

> ✅ Phase 2 Complete — PDC19 is now a fully operational Domain Controller for GBG.local.

---

## Phase 3 — Active Directory Structure and Entra Connect Sync

With the domain established, the AD structure is built: three OUs are created, users and security groups are provisioned inside the target OU, UPN suffixes are updated to match the Entra ID tenant domain, and Microsoft Entra Connect is installed and configured for scoped OU-level synchronization.

---

### Step 11 — Open Active Directory Users and Computers

ADUC is launched from Server Manager Tools. The GBG.local domain is visible with its default containers.

**Figure 16 — Server Manager Tools Menu: Launching Active Directory Users and Computers**
![Figure 16](images/image16.png)

---

### Step 12 — Create Organizational Units

Three OUs are created directly under GBG.local to reflect a realistic department structure: Security_Team, IT_Team, and HR_Team.

**Figure 17 — ADUC: Creating a New Organizational Unit Under GBG.local**
![Figure 17](images/image17.png)

**Figure 18 — ADUC: Three OUs Created — Security_Team, IT_Team, HR_Team**
![Figure 18](images/image18.png)

---

### Step 13 — Create Users Inside the Target OU

Users are created inside the Security_Team OU using the ADUC new user wizard. Three users are provisioned: Ahmed Elgohary, Basel Ali, and Clara Mohamed.

**Figure 19 — ADUC: Creating a New User Inside the Security_Team OU**
![Figure 19](images/image19.png)

**Figure 20 — Security_Team OU: Three Users and Two Security Groups Populated**
![Figure 20](images/image20.png)

---

### Step 14 — Create Security Groups and Assign Members

Two Global Security Groups are created inside Security_Team: SOC and GRC. Users are assigned to their respective groups via the Members tab.

**Figure 21 — SOC Group Properties: Members Tab Showing Assigned Users**
![Figure 21](images/image21.png)

**Figure 22 — GRC Group Properties: Members Tab Showing Assigned User**
![Figure 22](images/image22.png)

---

### Step 15 — Add Custom UPN Suffix in Active Directory Domains and Trusts

Before syncing to Entra ID, the cloud tenant domain `gbgacademy.online` must be added as a UPN suffix in AD. This is done via Active Directory Domains and Trusts → UPN Suffixes.

**Figure 23 — Active Directory Domains and Trusts: Adding gbgacademy.online as UPN Suffix**
![Figure 23](images/image23.png)

**Figure 24 — UPN Suffix gbgacademy.online Successfully Added to the Forest**
![Figure 24](images/image24.png)

**Figure 25 — Verifying UPN Suffix on a User Account: @gbgacademy.online Now Available**
![Figure 25](images/image25.png)

---

### Step 16 — Update All User UPNs via PowerShell

Rather than updating each user manually, a PowerShell script loops through all AD users and sets their UPN suffix to `@gbgacademy.online`. This is required for proper cloud identity matching during sync.

**Figure 26 — PowerShell Script: Bulk UPN Update for All AD Users to @gbgacademy.online**
![Figure 26](images/image26.png)

**Figure 27 — ADUC User Properties: UPN Successfully Updated to @gbgacademy.online**
![Figure 27](images/image27.png)

---

### Step 17 — Navigate to Entra Connect in the Azure Portal

The Entra ID portal is accessed from the DC's browser. Navigation goes to Identity → Hybrid Management → Microsoft Entra Connect → Connect Sync to locate the download link.

**Figure 28 — Azure Portal: Navigating to Microsoft Entra Connect — Connect Sync Section**
![Figure 28](images/image28.png)

**Figure 29 — Entra Connect Portal Page: Sync Status Enabled — Download Link Visible**
![Figure 29](images/image29.png)

---

### Step 18 — Activate Hybrid Identity Administrator Role via PIM

The account `khaled@gbgacademy.online` requires the Hybrid Identity Administrator role to configure Entra Connect. The role is activated through Privileged Identity Management (PIM) in the Entra Admin Center.

**Figure 30 — Entra Admin Center: PIM Role Activation for Hybrid Identity Administrator**
![Figure 30](images/image30.png)

**Figure 31 — PIM Activation Status: Hybrid Identity Administrator Role Successfully Activated**
![Figure 31](images/image31.png)

---

### Step 19 — Launch Entra Connect Installer on the Domain Controller

The `AzureADConnect.msi` installer is run on PDC19. The welcome screen confirms this is Microsoft Entra Connect Sync. The license agreement is accepted and Customize is selected to enable OU-level filtering.

**Figure 32 — Entra Connect Installer: Welcome Screen on the Domain Controller**
![Figure 32](images/image32.png)

**Figure 33 — Entra Connect: Express Settings Screen — Selecting Customize for OU Filtering**
![Figure 33](images/image33.png)

---

### Step 20 — Install Required Components

The required components screen is presented. No custom SQL server, service account, or sync groups are needed for this lab. All checkboxes are left empty and Install is clicked.

**Figure 34 — Entra Connect: Install Required Components — All Options Left as Default**
![Figure 34](images/image34.png)

---

### Step 21 — Select Authentication Method

The User Sign-In screen offers several authentication methods. Password Hash Synchronization is selected as it is the simplest and most suitable method for this lab environment.

**Figure 35 — Entra Connect: User Sign-In — Password Hash Synchronization Selected**
![Figure 35](images/image35.png)

---

### Step 22 — Connect to Microsoft Entra ID

The wizard prompts for Entra ID Global Admin or Hybrid Identity Admin credentials. The account `khaled@gbgacademy.online` is used, which now has the activated Hybrid Identity Administrator role.

**Figure 36 — Entra Connect: Connect to Microsoft Entra ID — Entering Cloud Admin Credentials**
![Figure 36](images/image36.png)

---

### Step 23 — Connect to On-Premises Active Directory

The on-premises AD forest `GBG.local` is detected. Enterprise Admin credentials `GBG\Administrator` are entered to allow Entra Connect to read and sync directory objects.

**Figure 37 — Entra Connect: Connect to AD DS — Entering On-Premises Admin Credentials**
![Figure 37](images/image37.png)

**Figure 38 — Entra Connect: AD Forest GBG.local Successfully Added and Verified**
![Figure 38](images/image38.png)

---

### Step 24 — Configure Domain and OU Filtering

This is the most critical step. Instead of syncing all OUs, only the `Security_Team` OU is selected. This ensures only the intended users and groups are synchronized to Entra ID.

**Figure 39 — Entra Connect: Domain and OU Filtering — Only Security_Team OU Selected**
![Figure 39](images/image39.png)

---

### Step 25 — Complete Configuration and Initiate Sync

The remaining wizard steps are accepted as default. The Ready to Configure screen summarizes all actions. The option to start synchronization immediately is checked, and Install is clicked.

**Figure 40 — Entra Connect: Ready to Configure — Summary of All Sync Settings**
![Figure 40](images/image40.png)

**Figure 41 — Entra Connect: Configuration Complete — Synchronization Process Initiated**
![Figure 41](images/image41.png)

---

### Step 26 — Trigger Manual Sync Cycle via PowerShell

After installation, a manual full sync is triggered from PowerShell to confirm the ADSync service is running and the initial sync cycle completes successfully.

**Figure 42 — PowerShell: ADSync Service Running and Initial Sync Cycle Result — Success**
![Figure 42](images/image42.png)

---

### Step 27 — Verify Synced Group in Entra ID Portal

The Entra ID Groups section is checked. The SOC group appears with Source listed as Windows Server AD, confirming it was synced from the on-premises domain and not created manually in the cloud.

**Figure 43 — Entra ID Portal: SOC Group Overview — Source: Windows Server AD**
![Figure 43](images/image43.png)

**Figure 44 — Entra ID Portal: SOC Group Overview Showing 2 Synced Members**
![Figure 44](images/image44.png)

---

### Step 28 — Verify Synced Users Inside the Group

The Members tab of the SOC group in Entra ID displays Ahmed Elgohary and Basel Ali — both synced from the on-premises Security_Team OU, with their group membership intact.

**Figure 45 — Entra ID Portal: SOC Group Members — Ahmed Elgohary and Basel Ali Synced**
![Figure 45](images/image45.png)

---

### Step 29 — Verify User On-Premises Sync Status in Entra ID

The All Users blade in Entra ID is reviewed. Synced users show "On-premises sync enabled = Yes", confirming their identity source is the on-premises Active Directory via Entra Connect.

**Figure 46 — Entra ID Portal: All Users List — On-Premises Sync Enabled for Synced Users**
![Figure 46](images/image46.png)

---

### Step 30 — Verify GRC Group Also Synced with Members

The GRC security group is also confirmed in Entra ID with its members synced from the on-premises AD, completing the verification of all objects from the Security_Team OU.

**Figure 47 — Entra ID Portal: GRC Group Members Verified After Sync**
![Figure 47](images/image47.png)

**Figure 48 — Entra ID Portal: Entra Connect Sync Status Showing Last Sync Timestamp**
![Figure 48](images/image48.png)

> ✅ Phase 3 Complete — All objects from the Security_Team OU (users, groups, and memberships) are successfully synchronized to Microsoft Entra ID via Entra Connect.

---

## Summary

| Task | Status |
|---|---|
| VM network configuration (Bridged + LAN Segment) | ✅ Complete |
| Static IP assignment on DC and Windows 10 | ✅ Complete |
| Domain Controller promotion (GBG.local) | ✅ Complete |
| OU structure creation (Security_Team, IT_Team, HR_Team) | ✅ Complete |
| User and group provisioning with membership assignment | ✅ Complete |
| UPN suffix update to @gbgacademy.online via PowerShell | ✅ Complete |
| Hybrid Identity Administrator role activation via PIM | ✅ Complete |
| Microsoft Entra Connect installation with OU-level filtering | ✅ Complete |
| Sync verification in Entra ID (users, groups, membership) | ✅ Complete |

---

## Key Takeaways

- **Scoped OU sync** prevents unnecessary cloud exposure of internal service accounts and administrative objects
- **UPN alignment** between on-prem AD and the Entra ID tenant domain is essential for identity matching during sync
- **PIM role activation** is the enterprise-standard approach to just-in-time privilege — roles are activated only when needed and expire automatically
- **Password Hash Sync** is the recommended starting point for hybrid identity — it provides cloud authentication resilience even if the on-premises DC is unreachable
- **Source of authority** remains on-premises AD for synced objects — changes must be made in AD, not in Entra ID

---

*Lab performed on VMware Workstation | Domain: GBG.local | Cloud Tenant: gbgacademy.online | Entra ID P2*
