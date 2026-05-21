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

<img width="975" height="859" alt="image" src="https://github.com/user-attachments/assets/ab187a9d-a3d7-4c6f-9936-5e01dd35b597" />


> In the Media Creation Tool, choose **Create installation media (USB flash drive, DVD, or ISO file) for another PC**, then click **Next**.

---

**Step 2 — Choose ISO file as the media type**

<img width="975" height="856" alt="image" src="https://github.com/user-attachments/assets/789c6c1b-748d-4bd0-b8c7-978fff782ff4" />


> Select **ISO file** and click **Next**. Choose a save location on your machine.

---

**Step 3 — ISO download complete**

<img width="975" height="609" alt="image" src="https://github.com/user-attachments/assets/24c6de39-896c-494e-bcfe-4b8b1ff18513" />
<img width="975" height="856" alt="image" src="https://github.com/user-attachments/assets/0b3a6018-6297-4ffd-aa00-4bd53722422f" />
<img width="975" height="870" alt="image" src="https://github.com/user-attachments/assets/91584f72-253b-4efa-8167-e807a3bf2099" />



> Once the download finishes, the tool confirms the ISO path. Click **Finish**.

---

**Step 4 — Note the ISO file path**


> Keep the ISO file path handy (e.g., `D:\Vm\ISOs\Windows 10\...`). You'll need it when configuring the VM.

---

### Part 2 — Create the Virtual Machine

**Step 5 — Launch VMware and create a new VM**

<img width="975" height="266" alt="image" src="https://github.com/user-attachments/assets/c0137de2-650c-4df8-a0e2-d92aba2998c2" />

> Open VMware Workstation Pro 17 and click **Create a New Virtual Machine**.

---

**Step 6 — Select "Typical" configuration**

<img width="628" height="663" alt="image" src="https://github.com/user-attachments/assets/db37cb1d-2c9b-465f-bba6-2566b4058c57" />

> Choose **Typical (recommended)** and click **Next**.

---

**Step 7 — Defer OS installation**

<img width="621" height="664" alt="image" src="https://github.com/user-attachments/assets/7a0dc979-dc25-4985-a7ef-32a842e59cb8" />

> Select **I will install the operating system later** so the VM is created with a blank disk. Click **Next**.

---

**Step 8 — Select Windows 10 x64 as the guest OS**

<img width="623" height="658" alt="image" src="https://github.com/user-attachments/assets/86c41e23-edfd-4a7f-86ea-9a175b1a4693" />

> Set the guest OS to **Microsoft Windows** and the version to **Windows 10 x64**. Click **Next**.

---

**Step 9 — Name the VM and set the save location**

<img width="614" height="653" alt="image" src="https://github.com/user-attachments/assets/b70e336a-89a0-43cd-ad27-55d800626160" />

> Give the VM a name (e.g., `Windows10`) and choose a storage path. Click **Next**.

---

**Step 10 — Set disk capacity to 100 GB**

<img width="634" height="675" alt="image" src="https://github.com/user-attachments/assets/c0afe3b1-1148-45d3-bb03-6901c6186fcc" />

> Set the maximum disk size to **100 GB** and choose **Split virtual disk into multiple files**. Click **Next**.

---

**Step 11 — Attach the ISO to the virtual CD/DVD drive**

<img width="975" height="978" alt="image" src="https://github.com/user-attachments/assets/f033465d-4f70-464c-8ea0-245866d5a862" />

> Open **VM Settings → CD/DVD (SATA)**, select **Use ISO image file**, browse to the downloaded ISO, and click **OK**.

---

### Part 3 — Install Windows 10

**Step 12 — Skip the product key**

<img width="975" height="603" alt="image" src="https://github.com/user-attachments/assets/7cd56440-8609-46c7-b242-2d851f41024b" />

> When prompted for a product key, click **I don't have a product key** to proceed with an unactivated install.

---

**Step 13 — Select Windows 10 Pro**



> Choose **Windows 10 Pro (x64)** from the edition list and click **Next**.

---

**Step 14 — Choose "Custom" installation**


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
