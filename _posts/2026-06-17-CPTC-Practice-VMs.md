---
title: Setting Up CPTC Practice VMs in Proxmox
date: 2026-06-17 00:00:00 CDT
categories: [Homelab, CPTC]
tags: [proxmox, cptc, vmdk, linux, windows]
---

Virtual machines from prior CPTC competitions can be found at [cptc.rit.edu](http://cptc.rit.edu/). This post covers how to import those VMs into Proxmox and recover credentials on both Linux and Windows machines.

## Importing a VMDK into Proxmox

Based off this [YouTube video](https://youtu.be/k6-miz1Tb80?si=9kusbzoy2lttllTX).

### 1. Create a Virtual Machine

- Give the VM a name
- On the **OS** tab, set to **"Do not use any media"**
- On the **Disk** tab, configure whatever you want — this disk will be deleted
- Set CPU, memory, and network as normal
- Click **Create**

### 2. Remove the Default Disk

1. Detach the drive from the VM
2. Once detached it will appear as an **unused disk** — remove it
3. The VM should now have no disk attached

### 3. Import the VMDK

Open the shell on the Proxmox node hosting the VM and download the VMDK file:

```bash
wget <url-to-vmdk>
```

Then import the disk using:

```bash
qm importdisk <vmid> <file> <storage>
```

Example:

```bash
qm importdisk 200 example.vmdk tank
```

> Replace `tank` with your storage pool name if it differs.

The disk will now appear as `vm-<vmid>-disk-0` under **VM Disks** in storage.

### 4. Mount the Imported Disk

Before mounting, ensure the **SCSI Controller** is set to **VirtIO SCSI**.

Double-click the unused disk to mount it:
- Set the bus device to **SCSI**
- Confirm the correct disk image is selected
- Click **Add**

To expand the disk, select it and use the **Resize Disk** option.

### 5. Set Boot Order

Go to **Options → Boot Order**, add the new drive, and drag it to the top.

The VM is now ready to start or clone as a template.

---

## Recovering Credentials on Linux

To reset the root password after importing a Linux VM:

1. Spam **Shift** during boot when the Ubuntu logo appears
2. Press **`e`** to edit GRUB commands
3. On the line beginning with `linux`, change `ro` to `rw` and remove any `console=` options
4. Append `init=/bin/bash` to the end of that line
5. Press **Ctrl+X** to boot — you should drop into a root shell
6. Remount the filesystem as read-write:
   ```bash
   mount -o remount,rw /
   ```
7. Change the root password:
   ```bash
   passwd
   ```
8. Reboot:
   ```bash
   reboot -f
   ```

---

## Recovering Credentials on Windows

To reset a forgotten administrator password on a Windows VM:

1. Boot from an ISO and open a **Command Prompt**
2. Navigate to System32:
   ```cmd
   cd /d C:\Windows\System32
   ```
3. Back up Utilman.exe and replace it with cmd.exe:
   ```cmd
   copy C:\Windows\System32\Utilman.exe C:\Windows\System32\Utilman.bak
   copy C:\Windows\System32\cmd.exe C:\Windows\System32\Utilman.exe
   ```
4. Boot the machine normally
5. On the login screen, click **Ease of Access** — this now opens a command prompt
6. Reset the administrator password:
   ```cmd
   net user administrator <password>
   ```
7. Once logged in, restore Utilman.exe:
   ```cmd
   takeown /f Utilman.exe
   icacls Utilman.exe /grant administrators:F
   copy Utilman.bak Utilman.exe
   ```
