# Hybrid Identity Lab — Active Directory to Microsoft Entra ID

> End-to-end implementation of a hybrid identity environment, bridging an on-premises Active Directory infrastructure with Microsoft Entra ID (formerly Azure AD), built entirely on VMware Workstation.

---

## Table of Contents

- [Introduction](#introduction)
- [Objectives](#objectives)
- [Tools & Technologies](#tools--technologies)
- [Conceptual Foundation](#conceptual-foundation)
- [Lab Implementation](#lab-implementation)
  - [Phase 1 — Network Configuration](#phase-1--network-configuration)
  - [Phase 2 — Domain Controller Promotion](#phase-2--domain-controller-promotion)
  - [Phase 3 — Active Directory Structure](#phase-3--active-directory-structure)
  - [Phase 4 — UPN Configuration & Entra Connect Installation](#phase-4--upn-configuration--entra-connect-installation)
  - [Phase 5 — Domain Join & Final Verification](#phase-5--domain-join--final-verification)

---

## Introduction

This lab documents the end-to-end implementation of a hybrid identity environment that bridges an on-premises Active Directory infrastructure with Microsoft Entra ID. The environment is built entirely on VMware Workstation using isolated virtual machines, simulating a real enterprise network topology.

The lab covers domain controller deployment, organizational unit design, user and group provisioning, identity synchronization via Microsoft Entra Connect, and full verification of synced objects in the cloud tenant.

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

## Conceptual Foundation

### 1. Microsoft 365 Group vs. Security Group

| Feature | Microsoft 365 Group | Security Group |
|---|---|---|
| Purpose | Collaboration (Teams, SharePoint, Exchange) | Access control and permissions |
| Has Mailbox | Yes | No |
| Used for resource permissions | Limited | Yes |
| Created in | Microsoft 365 / Entra ID | Active Directory or Entra ID |
| Synced from on-prem AD | No | Yes |
| Supports dynamic membership | Yes | Yes (Entra ID P1/P2) |

**In short:** M365 Groups are collaboration containers. Security Groups are used to grant or restrict access to resources.

---

### 2. Membership Types — Assigned, Dynamic User, Dynamic Group

**Assigned** — Membership is managed manually by an administrator. Users are added or removed explicitly with no automation involved.

**Dynamic User** — A user account whose attributes (such as department, job title, or location) automatically trigger group membership based on a defined rule. Example: `department -eq "IT"` — any user with IT as their department is automatically added to the group.

**Dynamic Group** — A group whose membership is entirely maintained by Entra ID based on attribute rules. No manual additions are needed; the engine evaluates rules continuously and adjusts membership as attributes change.

> Requires **Entra ID P1 or P2** license for dynamic membership rules.

---

### 3. Authentication Methods in Microsoft Entra ID

| Method | How It Works | Best For |
|---|---|---|
| **Password Hash Sync (PHS)** | AD password hashes are synced to Entra ID; authentication happens in the cloud | Simplest hybrid setup, high availability |
| **Pass-Through Authentication (PTA)** | Authentication request is forwarded to on-prem AD in real time; passwords never leave the network | Compliance-driven orgs that cannot store hashes in the cloud |
| **Federation (AD FS)** | Entra ID redirects auth to a federated identity provider; full SSO experience | Complex enterprise SSO with smart cards or MFA tokens |
| **Cloud-Only Authentication** | Users exist only in Entra ID with no on-prem dependency | Cloud-native organizations |
| **Microsoft Authenticator (MFA)** | Push notification, OTP, or passwordless sign-in via the app | MFA enforcement across all users |
| **FIDO2 Security Keys** | Hardware keys for passwordless authentication | High-security environments |
| **Windows Hello for Business** | Biometric or PIN-based authentication tied to device | Modern workplace with Hybrid or Entra-joined devices |

---

## Lab Implementation

---

### Phase 1 — Network Configuration

Before any services are configured, both virtual machines must have their network adapters set up correctly. The Windows Server receives two adapters: a Bridged adapter for internet access, and a LAN Segment adapter (LAN-5) for internal communication with the Windows 10 client. The client is then configured with a static IP pointing to the server as its default gateway and DNS.

---

#### Step 1.1 — Configuring VM Network Adapters (Server)

The Windows Server VM is configured with two network adapters. The first is a Bridged adapter providing internet connectivity. The second is a LAN Segment (LAN-5) adapter for internal network isolation.

---

![Figure 1](figures/figure01.png)

*Figure 1 — First network adapter configured as Bridged, connecting the server directly to the physical network*

---

![Figure 2](figures/figure02.png)

*Figure 2 — Second network adapter added and set to LAN-5, establishing the isolated internal segment*

---

#### Step 1.2 — Configuring Static IP on the LAN Segment Interface

Inside Server Manager under Local Server, the LAN-5 adapter (Ethernet 1) is assigned a static IP address of `192.168.10.1` with subnet mask `255.255.255.0`. The Bridged adapter (Ethernet 0) is left on DHCP for internet access.

---

![Figure 3](figures/figure03.png)

*Figure 3 — Server Manager Local Server overview prior to network configuration*

---

![Figure 4](figures/figure04.png)

*Figure 4 — Network Connections panel showing both adapters: the Bridged internet adapter and the LAN-5 internal adapter*

---

![Figure 5](figures/figure05.png)

*Figure 5 — Ethernet properties dialog open for the LAN-5 interface, ready for static IP assignment*

---

![Figure 6](figures/figure06.png)

*Figure 6 — Static IP address 192.168.10.1 assigned to the LAN-5 adapter with DNS set to loopback*

---

![Figure 7](figures/figure07.png)

*Figure 7 — Server Manager reflecting the updated LAN-5 IP address after successful static configuration*

---

#### Step 1.3 — Configuring the Windows 10 Client IP

The Windows 10 VM is configured with a static IP of `192.168.10.20`, pointing to the server (`192.168.10.1`) as both the default gateway and preferred DNS server.

---

![Figure 8](figures/figure08.png)

*Figure 8 — Windows 10 IPv4 properties configured with static IP 192.168.10.20 and gateway 192.168.10.1*

---

#### Step 1.4 — Verifying Connectivity

A ping test from the Windows 10 client to the server IP confirms successful bidirectional communication across the LAN segment with zero packet loss.

---

![Figure 9](figures/figure09.png)

*Figure 9 — Successful ping to 192.168.10.1 from the Windows 10 client, confirming network connectivity*

---

### Phase 2 — Domain Controller Promotion

With the network in place, the server is promoted to a Domain Controller under a new Active Directory forest named `GBG.local`. DNS and Global Catalog roles are installed alongside AD DS during this phase.

---

#### Step 2.1 — Triggering the AD DS Promotion Wizard

After installing the AD DS role, Server Manager displays a post-deployment configuration notification. Clicking "Promote this server to a domain controller" launches the configuration wizard.

---

![Figure 10](figures/figure10.png)

*Figure 10 — Server Manager post-deployment notification prompting domain controller promotion*

---

#### Step 2.2 — Creating a New Forest

In the wizard, "Add a new forest" is selected with `GBG.local` set as the root domain name.

---

![Figure 11](figures/figure11.png)

*Figure 11 — Deployment Configuration step with "Add a new forest" selected and root domain set to GBG.local*

---

#### Step 2.3 — Setting Domain Controller Options

The forest and domain functional levels are both set to Windows Server 2016. The DNS Server and Global Catalog options are enabled, and the DSRM password is configured.

---

![Figure 12](figures/figure12.png)

*Figure 12 — Domain Controller Options with DNS server and Global Catalog enabled, functional level set to Windows Server 2016*

---

#### Step 2.4 — Prerequisites Check & Installation

The wizard completes its prerequisites check successfully. The server is then promoted and automatically reboots to apply all changes.

---

![Figure 13](figures/figure13.png)

*Figure 13 — Prerequisites check passed with informational warnings; ready for installation*

---

![Figure 14](figures/figure14.png)

*Figure 14 — Server rebooting as part of the domain controller promotion process*

---

#### Step 2.5 — Verifying Domain Membership

After reboot, Server Manager confirms the server is now a member of `GBG.local` and the AD DS and DNS roles are fully operational.

---

![Figure 15](figures/figure15.png)

*Figure 15 — Server Manager Local Server showing domain joined as GBG.local, confirming successful promotion*

---

### Phase 3 — Active Directory Structure

With the domain controller running, the AD structure is built: Organizational Units are created to logically segment users, users are provisioned inside those OUs, and security groups are formed and populated.

---

#### Step 3.1 — Opening Active Directory Users and Computers

From Server Manager's Tools menu, Active Directory Users and Computers (ADUC) is launched to begin building the directory structure.

---

![Figure 16](figures/figure16.png)

*Figure 16 — Server Manager Tools menu with Active Directory Users and Computers highlighted*

---

#### Step 3.2 — Creating Organizational Units

Three Organizational Units are created under the GBG.local domain: `Security_Team`, `IT_Team`, and `HR_Team`. These OUs will serve as containers for users and will be the sync scope for Entra Connect.

---

![Figure 17](figures/figure17.png)

*Figure 17 — Right-click context menu in ADUC with the New > Organizational Unit option selected*

---

![Figure 18](figures/figure18.png)

*Figure 18 — ADUC tree view showing the three newly created OUs: Security_Team, IT_Team, and HR_Team*

---

#### Step 3.3 — Creating Users

Users are created inside the appropriate OUs. The user creation wizard captures first name, last name, and the UPN logon name.

---

![Figure 19](figures/figure19.png)

*Figure 19 — New user creation initiated inside the Security_Team OU*

---

![Figure 20](figures/figure20.png)

*Figure 20 — User creation form filled with details for Ahmed Elgohary, UPN set to Ahmed.Elgohary@GBG.local*

---

![Figure 21](figures/figure21.png)

*Figure 21 — Security_Team OU populated with three users: Ahmed Elgohary, Basel Ali, and Clara Mohamed*

---

#### Step 3.4 — Creating Security Groups

Two security groups — `SOC` and `GRC` — are created inside the `Security_Team` OU as Global Security Groups.

---

![Figure 22](figures/figure22.png)

*Figure 22 — New group creation initiated inside the Security_Team OU*

---

![Figure 23](figures/figure23.png)

*Figure 23 — Security_Team OU now containing three users alongside the SOC and GRC security groups*

---

#### Step 3.5 — Adding Members to Groups

Users are added to their respective groups. The SOC group receives Ahmed Elgohary, Basel Ali, and Khaled Elgohary. The GRC group receives Clara Mohamed.

---

![Figure 24](figures/figure24.png)

*Figure 24 — Member selection dialog used to add Clara Mohamed to the GRC group*

---

![Figure 25](figures/figure25.png)

*Figure 25 — SOC group properties showing three members: Ahmed Elgohary, Basel Ali, and Khaled Elgohary*

---

### Phase 4 — UPN Configuration & Entra Connect Installation

Before synchronization can occur, a routable UPN suffix must be added and applied to all on-premises user accounts. Entra Connect is then downloaded, installed, and configured to sync identities to the cloud tenant.

---

#### Step 4.1 — Adding the UPN Suffix

Using Active Directory Domains and Trusts, the alternative UPN suffix `gbgacademy.online` is added to the forest. This routable domain is verified in Entra ID and will be used as the sync UPN.

---

![Figure 26](figures/figure26.png)

*Figure 26 — Server Manager Dashboard with the Active Directory Domains and Trusts tool highlighted*

---

![Figure 27](figures/figure27.png)

*Figure 27 — Active Directory Domains and Trusts console connected to PDC19.GBG.local*

---

![Figure 28](figures/figure28.png)

*Figure 28 — Alternative UPN suffix gbgacademy.online added to the forest properties*

---

#### Step 4.2 — Bulk UPN Update via PowerShell

A PowerShell script updates the UPN of every user in the directory to use the new `@gbgacademy.online` suffix, enabling them to authenticate against the verified cloud domain after sync.

```powershell
Import-Module ActiveDirectory
$Users = Get-ADUser -Filter *
foreach ($User in $Users) {
    Set-ADUser -Identity $User -UserPrincipalName "$($User.SamAccountName)@gbgacademy.online"
}
```

---

![Figure 29](figures/figure29.png)

*Figure 29 — User properties reflecting the updated UPN suffix gbgacademy.online after PowerShell bulk update*

---

![Figure 30](figures/figure30.png)

*Figure 30 — ADUC confirming updated UPN on user accounts within the Security_Team OU*

---

#### Step 4.3 — Navigating to the Azure Portal

The Azure Portal is accessed at `https://portal.azure.com` to locate and download the Microsoft Entra Connect installer.

---

![Figure 31](figures/figure31.png)

*Figure 31 — Azure Portal home page showing Entra ID and available Azure services*

---

#### Step 4.4 — Locating Entra Connect in the Portal

In the Entra ID blade, Microsoft Entra Connect is found under the hybrid identity section. The latest Connect Sync version is downloaded from there.

---

![Figure 32](figures/figure32.png)

*Figure 32 — Microsoft Entra ID overview in the Azure Portal with Connect highlighted in the navigation*

---

![Figure 33](figures/figure33.png)

*Figure 33 — Entra Connect blade showing current sync status and the download option for the latest version*

---

![Figure 34](figures/figure34.png)

*Figure 34 — Recent download history showing AzureADConnect.msi ready for installation*

---

#### Step 4.5 — Activating the Hybrid Identity Administrator Role

Before installing Entra Connect, the Hybrid Identity Administrator role must be active. This is done through Privileged Identity Management (PIM) in the Entra admin center.

---

![Figure 35](figures/figure35.png)

*Figure 35 — PIM My Roles view listing eligible assignments including Hybrid Identity Administrator*

---

![Figure 36](figures/figure36.png)

*Figure 36 — Hybrid Identity Administrator role activation dialog with duration and justification fields*

---

![Figure 37](figures/figure37.png)

*Figure 37 — PIM processing the role activation request across three validation stages*

---

![Figure 38](figures/figure38.png)

*Figure 38 — Assigned roles view confirming Hybrid Identity Administrator is now actively assigned*

---

#### Step 4.6 — Installing Microsoft Entra Connect Sync

The Entra Connect installer is launched on the domain controller. Express settings are used with customization applied to accommodate the non-routable `GBG.local` domain.

---

![Figure 39](figures/figure39.png)

*Figure 39 — Microsoft Entra Connect Sync welcome screen with license agreement accepted*

---

![Figure 40](figures/figure40.png)

*Figure 40 — Express Settings page noting that GBG.local is non-routable and recommending custom settings*

---

#### Step 4.7 — Connecting to Entra ID and AD DS

The wizard prompts for cloud credentials (Hybrid Identity Administrator account) and then for on-premises AD enterprise administrator credentials to establish both connections.

---

![Figure 41](figures/figure41.png)

*Figure 41 — Cloud credential entry using the khaled@gbgacademy.online Hybrid Identity Administrator account*

---

![Figure 42](figures/figure42.png)

*Figure 42 — On-premises AD DS credential entry using the GBG\Administrator enterprise account*

---

#### Step 4.8 — UPN Suffix Mapping Verification

The wizard displays a UPN suffix mapping table. The `gbgacademy.online` suffix is shown as Verified while `gbg.local` appears as Not Added — confirming correct configuration.

---

![Figure 43](figures/figure43.png)

*Figure 43 — UPN suffix mapping table confirming gbgacademy.online as verified in Entra ID*

---

#### Step 4.9 — Completing Installation & Triggering Sync

The configuration summary is reviewed and installation is executed. After completion, an initial sync cycle is triggered manually via PowerShell to immediately push identities to Entra ID.

---

![Figure 44](figures/figure44.png)

*Figure 44 — Ready to Configure summary listing all actions Entra Connect will perform*

---

![Figure 45](figures/figure45.png)

*Figure 45 — PowerShell confirming ADSync service is running and initial sync cycle completed successfully*

---

### Phase 5 — Domain Join & Final Verification

The Windows 10 client is joined to the `GBG.local` domain and moved into the correct OU. Final verification is performed in both the Azure Portal and on the Windows 10 machine by signing in with synced hybrid credentials.

---

#### Step 5.1 — Verifying Synced Groups in Entra ID

The Azure Portal is checked to confirm that the SOC and GRC security groups have been successfully synced from on-premises AD to Entra ID.

---

![Figure 46](figures/figure46.png)

*Figure 46 — Entra ID All Groups view showing the SOC group synced from on-premises Active Directory*

---

![Figure 47](figures/figure47.png)

*Figure 47 — SOC group members in Entra ID: Ahmed Elgohary, Basel Ali, and Khaled — all confirmed as synced*

---

![Figure 48](figures/figure48.png)

*Figure 48 — GRC group members in Entra ID showing Clara Mohamed successfully synced from on-premises AD*

---

#### Step 5.2 — Joining the Windows 10 Client to the Domain

The Windows 10 machine is joined to `GBG.local` using System Properties. Enterprise administrator credentials are provided to authorize the join.

---

![Figure 49](figures/figure49.png)

*Figure 49 — Windows 10 System Properties showing domain join field with GBG.local entered*

---

![Figure 50](figures/figure50.png)

*Figure 50 — Domain join credential prompt requesting GBG\Administrator credentials*

---

![Figure 51](figures/figure51.png)

*Figure 51 — Confirmation dialog welcoming the machine to the GBG.local domain*

---

#### Step 5.3 — Moving the Computer Object into the OU

After the join, the computer object `KHALEDELGOHARY` appears in the default Computers container in ADUC. It is then moved into the `Security_Team` OU for proper organizational alignment.

---

![Figure 52](figures/figure52.png)

*Figure 52 — ADUC showing the KHALEDELGOHARY computer object in the default Computers container*

---

![Figure 53](figures/figure53.png)

*Figure 53 — Move dialog with Security_Team selected as the destination container*

---

![Figure 54](figures/figure54.png)

*Figure 54 — Security_Team OU containing all users, groups, and the domain-joined computer object*

---

#### Step 5.4 — Final Verification: Hybrid Identity in Action

The lab concludes with a side-by-side view confirming complete hybrid identity success: the Windows 10 machine is logged in as `Ahmed.Elgohary.GBG`, the Entra ID Users portal shows 66 synced users with on-premises identity markers, and the SOC group in Entra ID lists all correct members — all synced from on-premises AD.

---

![Figure 55](figures/figure55.png)

*Figure 55 — Final state: Windows 10 logged in with hybrid credentials, Entra ID reflecting 66 synced users, and SOC group membership fully verified in the cloud*

---

## Result

All on-premises Active Directory identities — users, groups, and the computer object — are successfully synchronized to Microsoft Entra ID. Users can now authenticate against both `GBG.local` and `gbgacademy.online`, and group membership is reflected accurately in the cloud tenant, completing the hybrid identity bridge between on-premises AD and Microsoft Entra ID.

---

> **Lab by:** Khaled Elgohary  
> **Domain:** GBG.local → gbgacademy.online  
> **Completed:** May 2026
