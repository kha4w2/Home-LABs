# Hybrid Identity Lab — Active Directory to Microsoft Entra ID

> End-to-end implementation of a hybrid identity environment, bridging an on-premises Active Directory infrastructure with Microsoft Entra ID (formerly Azure AD), built entirely on VMware Workstation.

<img width="1407" height="768" alt="Gemini_Generated_Image_eur6h2eur6h2eur6" src="https://github.com/user-attachments/assets/d400934e-e371-4eaa-9b62-05dbe37e5d4a" />
<img width="975" height="518" alt="image" src="https://github.com/user-attachments/assets/e71247f8-4119-41e7-a773-178c52fe3ded" />


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

<img width="975" height="980" alt="image" src="https://github.com/user-attachments/assets/8573e8c0-4220-407e-a19f-d9e1a8b4af35" />

*Figure 1 — First network adapter configured as Bridged, connecting the server directly to the physical network*

---

<img width="975" height="1003" alt="image" src="https://github.com/user-attachments/assets/f1e6431c-46db-4ac7-87f6-4bb4033fc99a" />

*Figure 2 — Second network adapter added and set to LAN-5, establishing the isolated internal segment*

---

#### Step 1.2 — Configuring Static IP on the LAN Segment Interface

Inside Server Manager under Local Server, the LAN-5 adapter (Ethernet 1) is assigned a static IP address of `192.168.10.1` with subnet mask `255.255.255.0`. The Bridged adapter (Ethernet 0) is left on DHCP for internet access.

---

<img width="975" height="480" alt="image" src="https://github.com/user-attachments/assets/248aac8b-658f-47bd-81e8-b6b3996fd782" />

*Figure 3 — Server Manager Local Server overview prior to network configuration*

---

<img width="975" height="505" alt="image" src="https://github.com/user-attachments/assets/6a0dfd45-6c6c-408a-9e7d-9ea080ffba68" />

*Figure 4 — Network Connections panel showing both adapters: the Bridged internet adapter and the LAN-5 internal adapter*

---

<img width="975" height="500" alt="image" src="https://github.com/user-attachments/assets/d815e6b6-6f87-4186-a53c-519b776b8187" />

*Figure 5 — Ethernet properties dialog open for the LAN-5 interface, ready for static IP assignment*

---

<img width="975" height="466" alt="image" src="https://github.com/user-attachments/assets/5ca33e8d-2116-4cc1-9de8-9b84c1d48c24" />

*Figure 6 — Static IP address 192.168.10.1 assigned to the LAN-5 adapter with DNS set to loopback*

---

<img width="975" height="506" alt="image" src="https://github.com/user-attachments/assets/e41c5c5e-b78c-48f0-b758-ce98763b28ee" />

*Figure 7 — Server Manager reflecting the updated LAN-5 IP address after successful static configuration*

---

#### Step 1.3 — Configuring the Windows 10 Client IP

The Windows 10 VM is configured with a static IP of `192.168.10.20`, pointing to the server (`192.168.10.1`) as both the default gateway and preferred DNS server.

---

<img width="975" height="509" alt="image" src="https://github.com/user-attachments/assets/2227d70a-7636-4bb3-abf1-6b7f6fbb5f9a" />

*Figure 8 — Windows 10 IPv4 properties configured with static IP 192.168.10.20 and gateway 192.168.10.1*

---

#### Step 1.4 — Verifying Connectivity

A ping test from the Windows 10 client to the server IP confirms successful bidirectional communication across the LAN segment with zero packet loss.

---

<img width="975" height="380" alt="image" src="https://github.com/user-attachments/assets/710450a1-3928-4f89-be92-8b724eba18ed" />

*Figure 9 — Successful ping to 192.168.10.1 from the Windows 10 client, confirming network connectivity*

---

### Phase 2 — Domain Controller Promotion

With the network in place, the server is promoted to a Domain Controller under a new Active Directory forest named `GBG.local`. DNS and Global Catalog roles are installed alongside AD DS during this phase.

---

#### Step 2.1 — Triggering the AD DS Promotion Wizard

After installing the AD DS role, Server Manager displays a post-deployment configuration notification. Clicking "Promote this server to a domain controller" launches the configuration wizard.

---

<img width="975" height="469" alt="image" src="https://github.com/user-attachments/assets/282de0ce-9f8f-490b-8e0e-d2f04e681e05" />

*Figure 10 — Server Manager post-deployment notification prompting domain controller promotion*

---

#### Step 2.2 — Creating a New Forest

In the wizard, "Add a new forest" is selected with `GBG.local` set as the root domain name.

---

<img width="975" height="470" alt="image" src="https://github.com/user-attachments/assets/95feecec-f20c-4a32-bdff-6b08b6f403ef" />

*Figure 11 — Deployment Configuration step with "Add a new forest" selected and root domain set to GBG.local*

---

#### Step 2.3 — Setting Domain Controller Options

The forest and domain functional levels are both set to Windows Server 2016. The DNS Server and Global Catalog options are enabled, and the DSRM password is configured.

---

<img width="975" height="464" alt="image" src="https://github.com/user-attachments/assets/a54fd241-199c-4875-b7b6-faed1cc9a026" />

*Figure 12 — Domain Controller Options with DNS server and Global Catalog enabled, functional level set to Windows Server 2016*

---

#### Step 2.4 — Prerequisites Check & Installation

The wizard completes its prerequisites check successfully. The server is then promoted and automatically reboots to apply all changes.

---

<img width="965" height="708" alt="image" src="https://github.com/user-attachments/assets/4aa2408b-4a39-49e6-a0d7-9b358bc8c6bc" />

*Figure 13 — Prerequisites check passed with informational warnings; ready for installation*

---

<img width="975" height="903" alt="image" src="https://github.com/user-attachments/assets/6dfe5f01-407e-4490-bf2c-40c27f9a0318" />

*Figure 14 — Server rebooting as part of the domain controller promotion process*

---

#### Step 2.5 — Verifying Domain Membership

After reboot, Server Manager confirms the server is now a member of `GBG.local` and the AD DS and DNS roles are fully operational.

---

<img width="975" height="465" alt="image" src="https://github.com/user-attachments/assets/e72d9cef-cf40-435c-97cf-6e98cfc35ed3" />

*Figure 15 — Server Manager Local Server showing domain joined as GBG.local, confirming successful promotion*

---

### Phase 3 — Active Directory Structure

With the domain controller running, the AD structure is built: Organizational Units are created to logically segment users, users are provisioned inside those OUs, and security groups are formed and populated.

---

#### Step 3.1 — Opening Active Directory Users and Computers

From Server Manager's Tools menu, Active Directory Users and Computers (ADUC) is launched to begin building the directory structure.

---

<img width="975" height="467" alt="image" src="https://github.com/user-attachments/assets/438605f3-d798-4324-9be6-0e195c5751f1" />

*Figure 16 — Server Manager Tools menu with Active Directory Users and Computers highlighted*

---

#### Step 3.2 — Creating Organizational Units

Three Organizational Units are created under the GBG.local domain: `Security_Team`, `IT_Team`, and `HR_Team`. These OUs will serve as containers for users and will be the sync scope for Entra Connect.

---

<img width="975" height="712" alt="image" src="https://github.com/user-attachments/assets/e1655dc2-dce0-42c5-88c7-8910ba44e5fe" />

*Figure 17 — Right-click context menu in ADUC with the New > Organizational Unit option selected*

---

<img width="975" height="570" alt="image" src="https://github.com/user-attachments/assets/6c8ccf8a-07c2-48fc-a78e-010268e8f2d9" />

*Figure 18 — ADUC tree view showing the three newly created OUs: Security_Team, IT_Team, and HR_Team*

---

#### Step 3.3 — Creating Users

Users are created inside the appropriate OUs. The user creation wizard captures first name, last name, and the UPN logon name.

---

<img width="975" height="723" alt="image" src="https://github.com/user-attachments/assets/d6311696-0c36-4596-bd2f-befe3bd85745" />

*Figure 19 — New user creation initiated inside the Security_Team OU*

---

<img width="675" height="578" alt="image" src="https://github.com/user-attachments/assets/27e4291a-31c5-4892-833f-40656c7e2bb6" />

*Figure 20 — User creation form filled with details for Ahmed Elgohary, UPN set to Ahmed.Elgohary@GBG.local*

---

<img width="928" height="489" alt="image" src="https://github.com/user-attachments/assets/670bfffe-c5a5-40ea-a688-cdcef92f1a6b" />

*Figure 21 — Security_Team OU populated with three users: Ahmed Elgohary, Basel Ali, and Clara Mohamed*

---

#### Step 3.4 — Creating Security Groups

Two security groups — `SOC` and `GRC` — are created inside the `Security_Team` OU as Global Security Groups.

---

<img width="941" height="489" alt="image" src="https://github.com/user-attachments/assets/41e1b9ad-9909-45b0-8c36-e024d20beac6" />

*Figure 22 — New group creation initiated inside the Security_Team OU*

---

<img width="975" height="493" alt="image" src="https://github.com/user-attachments/assets/f142a60a-a69b-477f-91d1-ef6eba8a62b7" />

*Figure 23 — Security_Team OU now containing three users alongside the SOC and GRC security groups*

---

#### Step 3.5 — Adding Members to Groups

Users are added to their respective groups. The SOC group receives Ahmed Elgohary, Basel Ali, and Khaled Elgohary. The GRC group receives Clara Mohamed.

---

<img width="703" height="388" alt="image" src="https://github.com/user-attachments/assets/0a63405f-15b6-4726-b795-c9ce090b1520" />

*Figure 24 — Member selection dialog used to add Clara Mohamed to the GRC group*

---

<img width="663" height="748" alt="image" src="https://github.com/user-attachments/assets/c3a5f41b-f099-486c-b622-96b346aab558" />

*Figure 25 — SOC group properties showing three members: Ahmed Elgohary, Basel Ali, and Khaled Elgohary*

---

### Phase 4 — UPN Configuration & Entra Connect Installation

Before synchronization can occur, a routable UPN suffix must be added and applied to all on-premises user accounts. Entra Connect is then downloaded, installed, and configured to sync identities to the cloud tenant.

---

#### Step 4.1 — Adding the UPN Suffix

Using Active Directory Domains and Trusts, the alternative UPN suffix `gbgacademy.online` is added to the forest. This routable domain is verified in Entra ID and will be used as the sync UPN.

---

<img width="975" height="446" alt="image" src="https://github.com/user-attachments/assets/45dd39a0-de77-454d-9e68-3e3b3b72e203" />

*Figure 26 — Server Manager Dashboard with the Active Directory Domains and Trusts tool highlighted*

---

<img width="975" height="663" alt="image" src="https://github.com/user-attachments/assets/56742d4b-9286-42e9-90fa-4c2ee2bce803" />

*Figure 27 — Active Directory Domains and Trusts console connected to PDC19.GBG.local*

---

<img width="631" height="714" alt="image" src="https://github.com/user-attachments/assets/ae51212b-8ddd-425e-b230-d0567e20a65a" />

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

<img width="975" height="379" alt="image" src="https://github.com/user-attachments/assets/3d2a0d36-b340-49b8-91cf-c3b44140b4be" />

*Figure 29 — User properties reflecting the updated UPN suffix gbgacademy.online after PowerShell bulk update*

---

<img width="975" height="463" alt="image" src="https://github.com/user-attachments/assets/51b8ab95-730b-4125-82bf-ec2c67700b72" />

*Figure 30 — ADUC confirming updated UPN on user accounts within the Security_Team OU*

---

#### Step 4.3 — Navigating to the Azure Portal

The Azure Portal is accessed at `https://portal.azure.com` to locate and download the Microsoft Entra Connect installer.

---

<img width="975" height="364" alt="image" src="https://github.com/user-attachments/assets/601ae825-0134-4d1b-b169-7f7a8ce12ce5" />

*Figure 31 — Azure Portal home page showing Entra ID and available Azure services*

---

#### Step 4.4 — Locating Entra Connect in the Portal

In the Entra ID blade, Microsoft Entra Connect is found under the hybrid identity section. The latest Connect Sync version is downloaded from there.

---

<img width="975" height="468" alt="image" src="https://github.com/user-attachments/assets/39852940-e9b6-4748-8a02-8195bbe923b5" />

*Figure 32 — Microsoft Entra ID overview in the Azure Portal with Connect highlighted in the navigation*

---

<img width="975" height="321" alt="image" src="https://github.com/user-attachments/assets/4de3c50f-ddbc-495e-94d9-fd335d8af45d" />

*Figure 33 — Entra Connect blade showing current sync status and the download option for the latest version*

---

<img width="975" height="157" alt="image" src="https://github.com/user-attachments/assets/e3ab29c9-4852-4e01-a0c5-ab9459475658" />

*Figure 34 — Recent download history showing AzureADConnect.msi ready for installation*

---

#### Step 4.5 — Activating the Hybrid Identity Administrator Role

Before installing Entra Connect, the Hybrid Identity Administrator role must be active. This is done through Privileged Identity Management (PIM) in the Entra admin center.

---

<img width="975" height="389" alt="image" src="https://github.com/user-attachments/assets/f10763eb-f6bc-4f7b-bbd3-31d900aeed87" />

*Figure 35 — PIM My Roles view listing eligible assignments including Hybrid Identity Administrator*

---

<img width="975" height="487" alt="image" src="https://github.com/user-attachments/assets/ea70bcf5-610e-4d91-a5d2-ed28994f7beb" />

*Figure 36 — Hybrid Identity Administrator role activation dialog with duration and justification fields*

---

<img width="975" height="479" alt="image" src="https://github.com/user-attachments/assets/0680d5ec-b9dd-4f22-bc99-dfde9115d054" />

*Figure 37 — PIM processing the role activation request across three validation stages*

---

<img width="975" height="345" alt="image" src="https://github.com/user-attachments/assets/08ff0b19-2cb4-4985-97c2-6d844f42a228" />

*Figure 38 — Assigned roles view confirming Hybrid Identity Administrator is now actively assigned*

---

#### Step 4.6 — Installing Microsoft Entra Connect Sync

The Entra Connect installer is launched on the domain controller. Express settings are used with customization applied to accommodate the non-routable `GBG.local` domain.

---

<img width="975" height="466" alt="image" src="https://github.com/user-attachments/assets/314cc6e6-ce09-45cf-9d40-e6ddd2c2134c" />

*Figure 39 — Microsoft Entra Connect Sync welcome screen with license agreement accepted*

---

<img width="975" height="506" alt="image" src="https://github.com/user-attachments/assets/25115809-4bf5-4294-b01c-d59327ea1e04" />

*Figure 40 — Express Settings page noting that GBG.local is non-routable and recommending custom settings*

---

#### Step 4.7 — Connecting to Entra ID and AD DS

The wizard prompts for cloud credentials (Hybrid Identity Administrator account) and then for on-premises AD enterprise administrator credentials to establish both connections.

---

<img width="975" height="679" alt="image" src="https://github.com/user-attachments/assets/fe19e096-1c61-4615-a845-769d5669cbd2" />

*Figure 41 — Cloud credential entry using the khaled@gbgacademy.online Hybrid Identity Administrator account*

---

<img width="975" height="681" alt="image" src="https://github.com/user-attachments/assets/30c73489-4fd0-4f0e-81f6-81bb99158f0b" />

*Figure 42 — On-premises AD DS credential entry using the GBG\Administrator enterprise account*

---

#### Step 4.8 — UPN Suffix Mapping Verification

The wizard displays a UPN suffix mapping table. The `gbgacademy.online` suffix is shown as Verified while `gbg.local` appears as Not Added — confirming correct configuration.

---

<img width="975" height="690" alt="image" src="https://github.com/user-attachments/assets/02b5dc57-d8c1-4833-b502-36dd07a944f8" />

*Figure 43 — UPN suffix mapping table confirming gbgacademy.online as verified in Entra ID*

---

#### Step 4.9 — Completing Installation & Triggering Sync

The configuration summary is reviewed and installation is executed. After completion, an initial sync cycle is triggered manually via PowerShell to immediately push identities to Entra ID.

---

<img width="975" height="693" alt="image" src="https://github.com/user-attachments/assets/a43987cf-ba30-4754-89ec-80ea2703e102" />

*Figure 44 — Ready to Configure summary listing all actions Entra Connect will perform*

---

<img width="823" height="459" alt="image" src="https://github.com/user-attachments/assets/1d777f5a-c33f-4623-9f88-6f24f697214b" />

*Figure 45 — PowerShell confirming ADSync service is running and initial sync cycle completed successfully*

---

### Phase 5 — Domain Join & Final Verification

The Windows 10 client is joined to the `GBG.local` domain and moved into the correct OU. Final verification is performed in both the Azure Portal and on the Windows 10 machine by signing in with synced hybrid credentials.

---

#### Step 5.1 — Verifying Synced Groups in Entra ID

The Azure Portal is checked to confirm that the SOC and GRC security groups have been successfully synced from on-premises AD to Entra ID.

---

<img width="975" height="276" alt="image" src="https://github.com/user-attachments/assets/3fce0c81-4205-4d8d-8b36-067970f2202c" />

*Figure 46 — Entra ID All Groups view showing the SOC group synced from on-premises Active Directory*

---

<img width="975" height="329" alt="image" src="https://github.com/user-attachments/assets/f4d9d786-d1a8-41f9-b788-f064ccfced81" />

*Figure 47 — SOC group members in Entra ID: Ahmed Elgohary, Basel Ali, and Khaled — all confirmed as synced*

---

<img width="975" height="272" alt="image" src="https://github.com/user-attachments/assets/c063c3e0-e4b5-4ffb-93f9-1fb9a5280909" />

*Figure 48 — GRC group members in Entra ID showing Clara Mohamed successfully synced from on-premises AD*

---

#### Step 5.2 — Joining the Windows 10 Client to the Domain

The Windows 10 machine is joined to `GBG.local` using System Properties. Enterprise administrator credentials are provided to authorize the join.

---

<img width="975" height="443" alt="image" src="https://github.com/user-attachments/assets/23836a86-20c5-4862-9d62-15b057e44d76" />

*Figure 49 — Windows 10 System Properties showing domain join field with GBG.local entered*

---

<img width="576" height="371" alt="image" src="https://github.com/user-attachments/assets/c7661a8b-eaf8-4438-b091-f9a690b900c6" />

*Figure 50 — Domain join credential prompt requesting GBG\Administrator credentials*

---

<img width="340" height="183" alt="image" src="https://github.com/user-attachments/assets/406e5874-0983-4cbd-9f38-50b8589c8e14" />

*Figure 51 — Confirmation dialog welcoming the machine to the GBG.local domain*

---

#### Step 5.3 — Moving the Computer Object into the OU

After the join, the computer object `KHALEDELGOHARY` appears in the default Computers container in ADUC. It is then moved into the `Security_Team` OU for proper organizational alignment.

---

<img width="975" height="534" alt="image" src="https://github.com/user-attachments/assets/1d075d4b-9ef9-4f59-ad36-edad595192a0" />

*Figure 52 — ADUC showing the KHALEDELGOHARY computer object in the default Computers container*

---

<img width="500" height="565" alt="image" src="https://github.com/user-attachments/assets/693631ff-da6b-472d-b869-4cbca0935d61" />

*Figure 53 — Move dialog with Security_Team selected as the destination container*

---

<img width="975" height="551" alt="image" src="https://github.com/user-attachments/assets/78e3ae12-3ab1-49b8-ac46-d6b2bc9f1c50" />

*Figure 54 — Security_Team OU containing all users, groups, and the domain-joined computer object*

---

#### Step 5.4 — Final Verification: Hybrid Identity in Action

The lab concludes with a side-by-side view confirming complete hybrid identity success: the Windows 10 machine is logged in as `Ahmed.Elgohary.GBG`, the Entra ID Users portal shows 66 synced users with on-premises identity markers, and the SOC group in Entra ID lists all correct members — all synced from on-premises AD.

---

<img width="975" height="518" alt="image" src="https://github.com/user-attachments/assets/e71247f8-4119-41e7-a773-178c52fe3ded" />

*Figure 55 — Final state: Windows 10 logged in with hybrid credentials, Entra ID reflecting 66 synced users, and SOC group membership fully verified in the cloud*

---

## Result

All on-premises Active Directory identities — users, groups, and the computer object — are successfully synchronized to Microsoft Entra ID. Users can now authenticate against both `GBG.local` and `gbgacademy.online`, and group membership is reflected accurately in the cloud tenant, completing the hybrid identity bridge between on-premises AD and Microsoft Entra ID.

---

> **Lab by:** Khaled Elgohary  
> **Domain:** GBG.local → gbgacademy.online  
> **Completed:** May 2026
