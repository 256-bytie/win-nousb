# Install Windows from Linux Without a USB Drive

**Boot any Windows ISO directly from your hard drive – no flash drive, no DVD, no external media.**

Works on **Arch Linux, Manjaro, Debian, Ubuntu, Fedora, Void Linux**, and most other distributions.

---

## 📋 Requirements

- Basic knowledge of the terminal
- A working Linux installation (UEFI mode – check with `ls /sys/firmware/efi`)
- At least **8 GB of free disk space** for the installer partition
- A separate **unallocated space** (or free partition) to install Windows onto
- A Windows ISO file (official or lightweight like tiny10)

> 💡 **Lightweight Windows ISO (tiny10)** is recommended because its `install.wim` is under 4 GB, so it fits easily on a FAT32 partition. Regular Windows ISOs work too but require splitting a large file.

---

## 🧱 Step 1 – Create a FAT32 partition for the installer

Use `gparted` (GUI) or `fdisk` / `parted` (terminal) to shrink an existing partition and create a new **FAT32** partition of **8–10 GB**.

### With GParted (easiest)
```bash
# Install gparted if needed
sudo pacman -S gparted        # Arch / Manjaro
sudo apt install gparted      # Debian / Ubuntu
sudo dnf install gparted      # Fedora

```
# **Launch gparted**
```bash
sudo gparted

```

# **Select your disk**
Usually `/dev/sda` or `/dev/nvme0n1`

• Then Right-click a partition with free space -> **Resize/Move**

• Shirley it to create **unallocated space** (at least 8gb)

• Right-click the unallocated space -> **New**

        ° Size: 8-10 GB
        ° File system: `fat32`
        
• Click the green checkmark to apply


**Result**: `/dev/sdaX` (e.g., `/dev/sda5`) appears, formatted as FAT32


> ⚠️  The partition must be FAT32 - UFI firmware only boots from FAT32.
  

# **Step 2 - Copy Windows ISO files to FAT32 partition**

Mount the FAT32 partition and the ISO, the copy everything.

```bash
# Create mount points
sudo mkdir -p /mnt/winusb /mnt/iso

# Mount the FAT32 partitions (replace X with your partition number)
sudo mount /dev/sdaX /mnt/winusb

# Mount the Windows ISO
sudo mount -o loop /path/to/windows.iso /mtn/iso

# Copy all files
sudo cp -r /mnt/iso/* /mnt/winusb/

# Clean up
sudo umount /mnt/iso
sudo umount /mnt/winusb
```

> ⚠️  **If you get an error about a file being too large (>4 GB)**

This happens with official Windows ISOs (the `sources/install.wim` file is often 5+ GB).

**Solution**: Split the large WIM file.

```bash
# Install wimlib
sudo pacman -S wimlib      # Arch
sudo apt install wimlib    # Debian
sudo dnf install wimlib    # Fedora

# After mounting the ISO, split install.wim
sudo wimlib-imagex split /mnt/iso/sources/install.wim /mnt/winusb/sources/install.swm 3800
```

Then copy the rest normally.


# **💻 Step 3 - Add a GRUB boot entry**

• **Edit the custom GRUB script**:

```bash
sudo nano /etc/grub.d/40_custom
```

• **Paste this**: 

```bash

#!/bin/sh
exec tail -n +3 $0

menuentry "Windows Installer" {
  insmod part_gpt
  insmod fat
  insmod chain
  search --no-floppy --set=root --file /efi/boot/bootx64.efi
  chainloader /efi/boot/bootx64.efi
}
```
Save: Ctrl+X -> Y -> Enter

• **Make the script executable and update GRUB**:

```bash
sudo chmod +x /etc/grub.d/40_custom
sudo update-grub
```

On **`Debian/Ubuntu`**, `update-grub` works. On **Fedora**, use:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```


# **Step 4 - Reboot and launch the Windows installer**

```bash
sudo reboot
```

During boot:

• Press `Shift` (or spam `ESC`) to show the GRUB menu
• Select "**Windows Installer**"
• Press Enter

# **Step 5 - Install Windows**

Do the usual.
