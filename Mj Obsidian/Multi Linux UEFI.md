✅ Perfect — since your **Lenovo IdeaPad Gaming 3 uses UEFI**, you’ll need to prepare your USB for **UEFI multiboot installation** of **multiple full Linux OSes**, each in its own partition.

Let’s now walk you through the **full step-by-step guide**, from partitioning to installing multiple Linux distros onto a **single USB**, using **VMware and UEFI boot**.

---

## 🧰 What You Need

|Item|Description|
|---|---|
|USB Drive|64GB minimum (128GB+ better)|
|VMware Workstation Player|To simulate installation|
|Linux ISOs (e.g., Ubuntu, Kali, Arch)|The distros to install|
|GParted Live ISO (optional)|For manual partitioning|

---

## 🗂️ Partition Plan for UEFI

UEFI systems **do not use BIOS boot or MBR**. You'll use **GPT partitioning** and an **EFI System Partition (ESP)**.

### Example Partition Layout:

|Partition|Type|Size|Format|Mount Point|Notes|
|---|---|---|---|---|---|
|EFI|Primary|512MB|FAT32|`/boot/efi`|Required for UEFI boot|
|Swap|Primary|4GB|swap|swap|Optional, can be shared|
|`/` root1|Logical|20GB|ext4|`/` (Ubuntu)|Root for first distro|
|`/` root2|Logical|20GB|ext4|`/` (Debian)|Root for second distro|
|`/` root3|Logical|20GB|ext4|`/` (Kali)|Root for third distro|
|`/home`|Logical|rest|ext4|(optional)|Optional shared data/home|

---

## 🧱 Step-by-Step Guide

---

### 🔧 Step 1: Create a New Virtual Machine (in VMware)

1. Launch **VMware Workstation Player**.
    
2. **Create New Virtual Machine**:
    
    - Choose: **Installer disc image file (ISO)** → Select Linux ISO (Ubuntu, etc.)
        
    - OS Type: Linux, Version: Ubuntu 64-bit (or similar)
        
    - Skip virtual disk creation (we’ll use USB drive)
        

---

### 🔌 Step 2: Attach Your USB Drive to VMware

1. Plug your USB drive into your PC.
    
2. Inside VMware, go to:
    
    - **Player → Removable Devices → [USB Drive] → Connect (Disconnect from Host)**
        

✅ Now the VM will use your real USB drive.

---

### 💽 Step 3: Boot into Live Linux Installer (e.g., Ubuntu)

1. Boot into the **Live ISO** (Try Ubuntu).
    
2. Open **GParted**:
    
    ```bash
    sudo gparted
    ```
    
3. Select your USB drive (e.g., `/dev/sdb`).
    
4. ⚠️ Delete all existing partitions.
    
5. Create a new **GPT partition table** (Device → Create Partition Table → GPT).
    

---

### 📐 Step 4: Partition Your USB Drive for UEFI Multiboot

Create the following partitions:

#### Partition 1: EFI System Partition

- Size: `512MB`
    
- Format: `FAT32`
    
- Set **"esp" and "boot" flags**
    
- Mount point: `/boot/efi`
    

#### Partition 2: Swap

- Size: `4GB`
    
- Format: `linux-swap`
    

#### Partition 3+: Logical ext4 Partitions for each Linux distro:

Create 20–30 GB ext4 partitions, one per OS. You can label them `ubuntu`, `kali`, etc.

💡 You can create more than 4 because **GPT doesn’t have the 4-primary limit like MBR**.

---

### 💻 Step 5: Install First Linux Distro (e.g., Ubuntu)

1. On the desktop, click **Install Ubuntu**.
    
2. Choose **"Something Else"** at the partitioning screen.
    
3. Assign partitions:
    
    - `/boot/efi` → EFI partition (don’t format if reused)
        
    - `/` → One of the root partitions
        
    - Swap → Swap partition
        
4. Select **USB device (e.g., `/dev/sdb`)** for bootloader installation.
    
5. Continue installation.
    

---

### 🔁 Step 6: Install Second and Third Distros

For each additional distro:

- Boot into the ISO in a **new VMware VM**, reusing the **same USB drive**.
    
- Use **"Something Else"** again.
    
- Set:
    
    - `/boot/efi` → same EFI partition (⚠️ Do not format again!)
        
    - `/` → different root partition
        
    - Swap → same swap
        
- Install GRUB to the **USB device** again (`/dev/sdb`)
    

💡 Only the **last-installed distro’s GRUB** will be active — it will control the boot menu.

---

### 🔄 Step 7: Update GRUB to Detect All Distros

After installing all OSes, boot into the **last-installed distro** on the USB, and run:

```bash
sudo update-grub
```

✅ This will detect all Linux OSes installed on the USB and add them to GRUB boot menu.

---

### 🔁 Optional: Manually Edit GRUB Entries

If one of the distros is not detected, you can manually edit:

```bash
sudo nano /etc/grub.d/40_custom
```

Add custom entries, then run:

```bash
sudo update-grub
```

---

### 🚀 Step 8: Boot the USB on Real Hardware

1. Shut down VMware.
    
2. Plug the USB into your real PC.
    
3. Power it on, and press **F12 / F10 / Esc** to open **Boot Menu**.
    
4. Choose the USB drive.
    
5. GRUB should appear with all your installed Linux OSes.
    

---

## ✅ Summary of Key Details

|Task|UEFI Version|
|---|---|
|Partitioning|Use GPT, create `/boot/efi` (FAT32, 512MB)|
|Bootloader Target|Always install GRUB to the USB device `/dev/sdX`|
|GRUB Sharing|EFI partition can be reused across all distros|
|Max Number of OSes|Unlimited (only space limits)|
|Swap Partition|Can be shared by all OSes|

---

## 🧠 Let Me Help You Further

If you tell me:

- Your USB size
    
- The Linux distros you want (e.g., Ubuntu, Kali, Fedora)
    
- Whether you want shared `/home` or data
    

…I’ll give you an exact partition table and GRUB setup plan tailored to you.

Ready to go?