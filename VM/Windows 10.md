# 🪟 Installing Windows 10 on VMware Workstation Pro 17

A concise, step-by-step guide to setting up a Windows 10 virtual machine using an ISO image and VMware Workstation Pro 17.

---

## 🎯 Objective

Download an official Windows 10 ISO via the Media Creation Tool, configure a new VM in VMware Workstation Pro 17, and perform a clean OS installation — all without a physical disc or USB drive.

---

## 📋 Prerequisites

- VMware Workstation Pro 17 installed
- At least 8 GB RAM and 100 GB free disk space on the host
- Internet connection to download the ISO

---

## 🛠️ Steps

### Part 1 — Download the Windows 10 ISO

**Step 1 — Select "Create installation media"**

![Select Create installation media](https://i.imgur.com/placeholder1.png)

> In the Media Creation Tool, choose **Create installation media (USB flash drive, DVD, or ISO file) for another PC**, then click **Next**.

---

**Step 2 — Choose ISO file as the media type**

![Choose ISO file](https://i.imgur.com/placeholder2.png)

> Select **ISO file** and click **Next**. Choose a save location on your machine.

---

**Step 3 — ISO download complete**

![ISO download complete](https://i.imgur.com/placeholder3.png)

> Once the download finishes, the tool confirms the ISO path. Click **Finish**.

---

**Step 4 — Note the ISO file path**

![Note the ISO file path](https://i.imgur.com/placeholder4.png)

> Keep the ISO file path handy (e.g., `D:\Vm\ISOs\Windows 10\...`). You'll need it when configuring the VM.

---

### Part 2 — Create the Virtual Machine

**Step 5 — Launch VMware and create a new VM**

![Create a new virtual machine in VMware](https://i.imgur.com/placeholder5.png)

> Open VMware Workstation Pro 17 and click **Create a New Virtual Machine**.

---

**Step 6 — Select "Typical" configuration**

![Select Typical configuration](https://i.imgur.com/placeholder6.png)

> Choose **Typical (recommended)** and click **Next**.

---

**Step 7 — Defer OS installation**

![Install OS later](https://i.imgur.com/placeholder7.png)

> Select **I will install the operating system later** so the VM is created with a blank disk. Click **Next**.

---

**Step 8 — Select Windows 10 x64 as the guest OS**

![Select Windows 10 x64](https://i.imgur.com/placeholder8.png)

> Set the guest OS to **Microsoft Windows** and the version to **Windows 10 x64**. Click **Next**.

---

**Step 9 — Name the VM and set the save location**

![Name the VM](https://i.imgur.com/placeholder9.png)

> Give the VM a name (e.g., `Windows10`) and choose a storage path. Click **Next**.

---

**Step 10 — Set disk capacity to 100 GB**

![Set disk capacity](https://i.imgur.com/placeholder10.png)

> Set the maximum disk size to **100 GB** and choose **Split virtual disk into multiple files**. Click **Next**.

---

**Step 11 — Attach the ISO to the virtual CD/DVD drive**

![Attach the ISO](https://i.imgur.com/placeholder11.png)

> Open **VM Settings → CD/DVD (SATA)**, select **Use ISO image file**, browse to the downloaded ISO, and click **OK**.

---

### Part 3 — Install Windows 10

**Step 12 — Skip the product key**

![Skip product key](https://i.imgur.com/placeholder12.png)

> When prompted for a product key, click **I don't have a product key** to proceed with an unactivated install.

---

**Step 13 — Select Windows 10 Pro**

![Select Windows 10 Pro](https://i.imgur.com/placeholder13.png)

> Choose **Windows 10 Pro (x64)** from the edition list and click **Next**.

---

**Step 14 — Choose "Custom" installation**

![Choose Custom installation](https://i.imgur.com/placeholder14.png)

> Select **Custom: Install Windows only (advanced)** for a clean install.

---

**Step 15 — Select the unallocated drive**

![Select unallocated drive](https://i.imgur.com/placeholder15.png)

> Select **Drive 0 Unallocated Space** and click **Next** to begin partitioning and installation.

---

**Step 16 — Windows installation in progress**

![Installation in progress](https://i.imgur.com/placeholder16.png)

> Setup copies files, installs features and updates. This takes several minutes — no action needed.

---

**Step 17 — Automatic restart**

![Automatic restart](https://i.imgur.com/placeholder17.png)

> Windows restarts automatically to finalize the installation. Let it complete.

---

**Step 18 — Windows 10 desktop ready**

![Windows 10 desktop](https://i.imgur.com/placeholder18.png)

> The Windows 10 desktop loads — your VM is up and running. 🎉

---

## ✅ Result

A fully functional Windows 10 Pro VM running inside VMware Workstation Pro 17, installed from an official Microsoft ISO with no USB or DVD required.

---

## 📝 Notes

- The VM was configured with **2 vCPUs**, **2 GB RAM**, and a **100 GB NVMe virtual disk**. Adjust to match your host hardware.
- The installation shown uses the **25H2 build** of Windows 10.
- You can activate Windows later via **Settings → Update & Security → Activation**.
