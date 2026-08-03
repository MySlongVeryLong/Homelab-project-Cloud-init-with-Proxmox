# Homelab-project-Cloud-init-with-Proxmox
idk, just a fun project , I did in my free time

In this project, I will go over how to provision VMs and instances on Proxmox using Cloud-Init to automate setting up the hostname, default user, password, and initial SSH key.

> **What is Cloud-Init?**

Cloud-init is a software package that can be installed on almost any operating system. It automates the initialization and configuration of virtual machines (VM) during the first boot, so we can define our hostname, configure network, DNS server, NTP server and so on.

### Requirements:
---
- Proxmox VE Server

## **Creating a Base Template with Cloud-Init**

In this section, I will use the **Ubuntu 24.04 LTS Cloud Image** for our template and **Cloud-Init** to pre-configure system settings.

### Steps:

1. Download the [Ubuntu Cloud Image](https://cloud-images.ubuntu.com/noble/current/) onto our Proxmox server.

![Ubuntu Cloud Image Download](images/Pasted%20image%2020260802220237.png)

2. Open the Proxmox terminal and create a VM shell using the following command:

![[Pasted image 20260802221955.png]]

> This command creates a VM named `Ubuntu-template` with ID `9000`, allocates `2G` of RAM, and sets up a network interface using the VirtIO driver.

3. Import the downloaded Ubuntu disk image into VM ID 9000 using `qcow2` as the disk format.

![[Pasted image 20260802222314.png]]

> **Note:** I placed this disk image in `local` storage rather than `local-lvm` due to storage pool constraints. Utilizing `local` with `qcow2` allows for thin provisioning and snapshot capabilities.

4. Attach a VirtIO SCSI controller and map the imported `qcow2` drive on `local` storage as `scsi0`.

![[Pasted image 20260802223320.png]]

> **Note:** I specified **`local:9000/vm-9000-disk-0.qcow2`** because `local` is directory-based storage and requires the subfolder path (`9000/`). If we were using `local-lvm` (block storage), specifying the subfolder path would not be necessary.

- In the Proxmox UI under our `Ubuntu-template` VM, we can verify that the SCSI controller, SCSI disk, and network devices are properly attached.

![[Pasted image 20260802224125.png]]

5. Attach the Cloud-Init drive.

![[Pasted image 20260802224603.png]]

> This attaches a virtual CD-ROM to our VM so Cloud-Init can feed configuration data to the OS on boot.

6. Configure the boot order to prioritize `scsi0`.

![[Pasted image 20260802225820.png]]

> Restricting the boot order to `scsi0` forces the virtual BIOS to boot directly into the OS drive, skipping the probe for a bootable CD-ROM on the Cloud-Init drive and significantly improving boot speed.

7. Configure a serial console device to display boot outputs directly from the Proxmox web shell.

![[Pasted image 20260802225127.png]]

Now that all necessary devices are attached, we can navigate to the Cloud-Init tab in the Proxmox UI to define initial credentials and network options.

![[Pasted image 20260802232057.png]]

Finally, right-click the VM, select **Convert to Template**, and start cloning automated instances in seconds.

![[Pasted image 20260802232336.png]]

![[Pasted image 20260802233212.png]]

