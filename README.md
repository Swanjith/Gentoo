# Gentoo Installation Guide:

HandBook Link:(https://wiki.gentoo.org/wiki/Handbook:AMD64)

## Table of Contents

1.  [Introduction](#introduction)
2.  [Choosing the Installation Media](#choosing-the-installation-media)
3.  [Configuring the Network](#configuring-the-network)
4.  [Preparing the Disks](#preparing-the-disks)
5.  [Installing the Gentoo Installation Files (Stage 3)](#installing-the-gentoo-installation-files-stage-3)
6.  [Installing the Gentoo Base System](#installing-the-gentoo-base-system)
7.  [Configuring the Linux Kernel](#configuring-the-linux-kernel)
8.  [Configuring the System](#configuring-the-system)
9.  [Installing System Tools](#installing-system-tools)
10. [Configuring the Bootloader](#configuring-the-bootloader)
11. [Finalizing the Installation](#finalizing-the-installation)

## 1. Introduction

Welcome to the world of Gentoo! Gentoo is a powerful and flexible Linux distribution that allows for deep customization and optimization. This guide will walk you through the process of installing Gentoo on your system, focusing on the AMD64 architecture.

Gentoo's core principles are **openness**, **choice**, and **power**. You'll be compiling most of your system from source, giving you unparalleled control over your operating system.

The installation process can be broken down into 10 main steps:

| Step | Result |
|---|---|
| 1 | The user is in a working environment ready to install Gentoo. |
| 2 | The Internet connection is ready to install Gentoo. |
| 3 | The hard disks are initialized to host the Gentoo installation. |
| 4 | The installation environment is prepared and the user is ready to chroot into the new environment. |
| 5 | Core packages, which are the same on all Gentoo installations, are installed. |
| 6 | The Linux kernel is installed. |
| 7 | Most of the Gentoo system configuration files are created. |
| 8 | The necessary system tools are installed. |
| 9 | The proper boot loader has been installed and configured. |
| 10 | The freshly installed Gentoo Linux environment is ready to be explored. |

Let's begin!




## 2. Choosing the Installation Media

To begin your Gentoo installation, you'll need to choose and prepare your installation media. The most common and recommended method is to use the **Minimal Installation CD**.

### Hardware Requirements

Before you start, ensure your system meets the minimum hardware requirements for AMD64:

*   **CPU:** Any x86-64 CPU (AMD64 or Intel 64)
*   **Memory:** 2 GB
*   **Disk space:** 8 GB (excluding swap space)
*   **Swap space:** At least 2 GB

### Obtaining the Minimal Installation CD

1.  **Download the ISO:** You can download the latest Minimal Installation CD ISO from the [official Gentoo downloads page](https://www.gentoo.org/downloads/). Alternatively, you can browse a [Gentoo mirror](https://www.gentoo.org/downloads/mirrors/) manually.

    If you choose a mirror, navigate to `releases/amd64/autobuilds/current-install-amd64-minimal/` to find the latest ISO image. The file will be named something like `install-amd64-minimal-<timestamp>Z.iso`.

2.  **Verify the Downloaded File:** It's crucial to verify the integrity of the downloaded ISO to ensure it hasn't been corrupted during download. You can do this using the `.DIGESTS` file and the `.asc` file (for GPG verification) found alongside the ISO on the mirror.

    **Example verification using `sha256sum` (Linux):**

    ```bash
    # Download the ISO and the .DIGESTS file
    wget https://<mirror>/releases/amd64/autobuilds/current-install-amd64-minimal/install-amd64-minimal-<timestamp>Z.iso
    wget https://<mirror>/releases/amd64/autobuilds/current-install-amd64-minimal/install-amd64-minimal-<timestamp>Z.iso.DIGESTS

    # Extract the SHA256 sum from the DIGESTS file
    grep SHA256 install-amd64-minimal-<timestamp>Z.iso.DIGESTS

    # Calculate the SHA256 sum of your downloaded ISO
    sha256sum install-amd64-minimal-<timestamp>Z.iso

    # Compare the two sums. They should match exactly.
    ```

    **Example GPG verification (Linux):**

    ```bash
    # Download the .asc file
    wget https://<mirror>/releases/amd64/autobuilds/current-install-amd64-minimal/install-amd64-minimal-<timestamp>Z.iso.asc

    # Import Gentoo's release key (if you haven't already)
    gpg --keyserver hkps.pool.sks-keyservers.net --recv-keys 0xBB572E0E2D182910

    # Verify the signature
    gpg --verify install-amd64-minimal-<timestamp>Z.iso.asc install-amd64-minimal-<timestamp>Z.iso
    ```

### Writing the Boot Media

Once you have the verified ISO, you need to write it to a USB drive or burn it to a DVD.

#### Writing to a USB Drive (Linux)

This is the most common method. Replace `/dev/sdX` with your USB drive's device path (e.g., `/dev/sdb`). **Be extremely careful to select the correct device, as this will erase all data on it.**

1.  **Determine your USB device path:**

    ```bash
    lsblk
    # or
    fdisk -l
    ```

    Look for your USB drive based on its size. For example, if your USB drive is 8GB, it might appear as `/dev/sdb`.

2.  **Write the ISO to the USB drive using `dd`:**

    ```bash
    sudo dd if=install-amd64-minimal-<timestamp>Z.iso of=/dev/sdX bs=1M status=progress
    sync
    ```

    This command will write the ISO image to your USB drive. `status=progress` will show you the progress of the operation.

#### Burning to a DVD (Linux)

If you prefer to use a DVD, you can burn the ISO using `wodim` or `growisofs`.

```bash
# Using wodim
sudo wodim dev=/dev/sr0 -v -data install-amd64-minimal-<timestamp>Z.iso

# Using growisofs
sudo growisofs -dvd-compat -Z /dev/sr0=install-amd64-minimal-<timestamp>Z.iso
```

### Booting the Installation Media

After preparing your bootable media, insert it into your computer and boot from it. You may need to adjust your BIOS/UEFI settings to boot from the USB drive or DVD.

Once booted, you'll be presented with a boot prompt. You can usually press Enter to boot with the default options, or choose specific kernel options if needed.

Next, we'll configure the network to download the necessary files for the installation.




## 3. Configuring the Network

An active internet connection is essential for installing Gentoo, as most of the installation involves downloading packages and source code. The Gentoo Minimal Installation CD usually attempts to configure your network automatically using DHCP. If you're connected via Ethernet to a network with a DHCP server, it might just work out of the box.

### Automatic Network Configuration (DHCP)

If your network uses DHCP, the system should automatically obtain an IP address. You can check if `dhcpcd` is running and obtain an IP address by running:

```bash
# If not already running, start dhcpcd on your Ethernet interface (e.g., enp1s0)
dhcpcd enp1s0
```

### Testing Network Connectivity

After booting and (if necessary) starting `dhcpcd`, you should test your network connection. 

1.  **Check your default route:**

    ```bash
    ip route
    ```

    You should see a line similar to `default via 192.168.0.1 dev enp1s0` (your IP and interface name may vary). If no default route is defined, you don't have internet connectivity yet.

2.  **Ping a well-known IP address:**

    ```bash
    ping -c 3 1.1.1.1
    ```

    This tests basic internet connectivity. If this works but `ping google.com` doesn't, you might have a DNS issue.

3.  **Test HTTPS access and DNS resolution:**

    ```bash
    curl --location gentoo.org --output /dev/null
    ```

    If `curl` reports no errors, your network is configured correctly, and you can proceed. If `curl` fails but `ping 1.1.1.1` works, you might need to configure DNS manually.

### Manual Network Configuration

If automatic configuration fails, you might need to configure your network manually.

#### Obtaining Interface Information

First, identify your network interfaces:

```bash
ip link
# Example output:
# 1: lo: <LOOPBACK,UP,LOWER_UP> ...
# 2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
```

Then, check their address information:

```bash
ip address
```

Note the name of your primary network interface (e.g., `enp1s0`).

#### Static IP Configuration

If you need to set a static IP address, you'll use the `ip` command. Replace `enp1s0`, `192.168.1.10/24`, and `192.168.1.1` with your actual interface name, IP address/netmask, and gateway.

```bash
# Configure IP address and netmask
ip addr add 192.168.1.10/24 dev enp1s0

# Set default route
ip route add default via 192.168.1.1

# Bring up the interface
ip link set enp1s0 up
```

#### DNS Configuration

If you have DNS issues, you can manually edit `/etc/resolv.conf` to add nameservers. For example, to use Google's public DNS:

```bash
echo 'nameserver 8.8.8.8' > /etc/resolv.conf
echo 'nameserver 8.8.4.4' >> /etc/resolv.conf
```

Once your network is configured and tested, you're ready to prepare your disks for the Gentoo installation.




## 4. Preparing the Disks

Before installing Gentoo, you need to prepare your disk(s) by creating partitions and filesystems. This section will guide you through the process, covering both UEFI (GPT) and Legacy BIOS (MBR) systems.

### Understanding Block Devices and Partition Tables

*   **Block Devices:** Your hard drives or SSDs are represented as block devices (e.g., `/dev/sda`, `/dev/sdb`, or `/dev/nvme0n1`).
*   **Partition Tables:** These define how your disk is divided into partitions. The two main types are:
    *   **GUID Partition Table (GPT):** Modern, recommended for UEFI systems. Supports larger disks and more partitions.
    *   **Master Boot Record (MBR):** Older, used with Legacy BIOS systems. Limited to 4 primary partitions.

### Designing a Partition Scheme

A common partitioning scheme for Gentoo includes:

*   **EFI System Partition (ESP):** (For UEFI systems) A small FAT32 partition (e.g., 256-512MB) for bootloader files.
*   **Swap Partition:** Used as virtual memory. A good rule of thumb is 1-2x your RAM, or at least 2GB.
*   **Root Partition (`/`):** The main partition where your Gentoo system will reside. This should be the largest partition.

### Partitioning the Disk with GPT for UEFI Systems

This is the recommended method for modern systems.

1.  **Identify your disk:** Use `lsblk` or `fdisk -l` to identify your target disk (e.g., `/dev/sda`).

    ```bash
    lsblk
    ```

2.  **Start `parted`:**

    ```bash
    sudo parted /dev/sda
    ```

3.  **Create a new GPT label:** This will erase all existing data on the disk.

    ```bash
    (parted) mklabel gpt
    ```

4.  **Create the EFI System Partition (ESP):** (e.g., 512MB, FAT32, boot flag)

    ```bash
    (parted) mkpart primary fat32 1MiB 513MiB
    (parted) set 1 boot on
    ```

5.  **Create the Swap Partition:** (e.g., 4GB)

    ```bash
    (parted) mkpart primary linux-swap 513MiB 4609MiB
    ```

6.  **Create the Root Partition:** Use the remaining space.

    ```bash
    (parted) mkpart primary ext4 4609MiB 100%
    ```

7.  **Verify the partition layout:**

    ```bash
    (parted) print
    ```

8.  **Exit `parted`:**

    ```bash
    (parted) quit
    ```

### Partitioning the Disk with MBR for Legacy BIOS Systems

Use this method if your system does not support UEFI or you prefer Legacy BIOS.

1.  **Identify your disk:** Use `lsblk` or `fdisk -l` to identify your target disk (e.g., `/dev/sda`).

    ```bash
    lsblk
    ```

2.  **Start `fdisk`:**

    ```bash
    sudo fdisk /dev/sda
    ```

3.  **Create a new DOS label:** This will erase all existing data on the disk.

    ```bash
    (fdisk) o
    ```

4.  **Create the Boot Partition:** (e.g., 256MB, primary, bootable)

    ```bash
    (fdisk) n
    (fdisk) p
    (fdisk) 1
    (fdisk) <Enter> (for default first sector)
    (fdisk) +256M
    (fdisk) a
    (fdisk) 1
    ```

5.  **Create the Swap Partition:** (e.g., 4GB)

    ```bash
    (fdisk) n
    (fdisk) p
    (fdisk) 2
    (fdisk) <Enter>
    (fdisk) +4G
    (fdisk) t
    (fdisk) 2
    (fdisk) 82 (Linux swap / Solaris)
    ```

6.  **Create the Root Partition:** Use the remaining space.

    ```bash
    (fdisk) n
    (fdisk) p
    (fdisk) 3
    (fdisk) <Enter>
    (fdisk) <Enter>
    ```

7.  **Verify the partition layout:**

    ```bash
    (fdisk) p
    ```

8.  **Write changes and exit `fdisk`:**

    ```bash
    (fdisk) w
    ```

### Creating File Systems

After partitioning, you need to format the partitions with appropriate file systems.

*   **For UEFI (GPT) systems:**

    ```bash
    # Format ESP (EFI System Partition) as FAT32
    sudo mkfs.fat -F 32 /dev/sda1

    # Format Swap partition
    sudo mkswap /dev/sda2

    # Format Root partition as Ext4
    sudo mkfs.ext4 /dev/sda3
    ```

*   **For Legacy BIOS (MBR) systems:**

    ```bash
    # Format Boot partition as Ext4
    sudo mkfs.ext4 /dev/sda1

    # Format Swap partition
    sudo mkswap /dev/sda2

    # Format Root partition as Ext4
    sudo mkfs.ext4 /dev/sda3
    ```

### Activating Swap

Activate your swap partition:

```bash
sudo swapon /dev/sdaX # Replace X with your swap partition number
```

### Mounting the Root Partition

Create the mount point for your Gentoo installation and mount the root partition:

```bash
sudo mkdir --parents /mnt/gentoo
sudo mount /dev/sdaX /mnt/gentoo # Replace X with your root partition number
```

If you created separate partitions for `/boot`, `/home`, etc., mount them now:

```bash
sudo mkdir /mnt/gentoo/boot
sudo mount /dev/sdaY /mnt/gentoo/boot # Replace Y with your boot partition number

# Example for /home
sudo mkdir /mnt/gentoo/home
sudo mount /dev/sdaZ /mnt/gentoo/home # Replace Z with your home partition number
```

Now that your disks are prepared and mounted, we can proceed to installing the Gentoo installation files.




## 5. Installing the Gentoo Installation Files (Stage 3)

The core of your Gentoo system begins with the Stage 3 tarball. This archive contains a minimal Gentoo environment that will be expanded upon during the installation process.

### Choosing a Stage File

Stage files are pre-built archives that serve as the foundation for your Gentoo installation. They are generated based on specific profiles, which define the default settings and package sets for your system.

*   **OpenRC vs. systemd:** Gentoo supports both OpenRC (its native init system) and systemd. Choose the one that aligns with your preferences. For most new users, OpenRC is a good starting point.
*   **Multilib vs. No-multilib:**
    *   **Multilib:** (Recommended for AMD64) Allows for both 64-bit and 32-bit libraries, providing greater compatibility with a wider range of software.
    *   **No-multilib:** A pure 64-bit environment. Only recommended for advanced users with specific needs, as migrating to multilib later can be complex.

### Downloading the Stage File

1.  **Navigate to your mount point:**

    ```bash
    cd /mnt/gentoo
    ```

2.  **Set the Date and Time:** Accurate system time is crucial for secure downloads (especially with HTTPS). You can set it automatically using `chronyd` (if available on your installation media) or manually.

    *   **Automatic (recommended):**

        ```bash
        chronyd -q
        ```

    *   **Manual:**

        ```bash
        # Example: Set date to October 3rd, 2021, 13:16
        date 100313162021
        ```

3.  **Download the Stage 3 Tarball:** Obtain the URL for the latest `stage3-amd64-<date>.tar.xz` file from the [official Gentoo downloads page](https://www.gentoo.org/downloads/) or a [Gentoo mirror](https://www.gentoo.org/downloads/mirrors/). Look under `releases/amd64/autobuilds/current-stage3-amd64/`.

    ```bash
    wget https://<mirror>/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-<date>.tar.xz
    ```

4.  **Verify the Downloaded File:** Just like with the installation media, verify the integrity of the downloaded stage file using its `.DIGESTS` and `.asc` files.

    ```bash
    # Example verification using sha512sum
    wget https://<mirror>/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-<date>.tar.xz.DIGESTS
    grep SHA512 stage3-amd64-<date>.tar.xz.DIGESTS
    sha512sum stage3-amd64-<date>.tar.xz
    # Compare the sums

    # Example GPG verification
    wget https://<mirror>/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-<date>.tar.xz.asc
    gpg --verify stage3-amd64-<date>.tar.xz.asc stage3-amd64-<date>.tar.xz
    ```

### Installing the Stage File

Extract the stage3 tarball into your mounted Gentoo directory:

```bash
# Extract the stage3 tarball
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

### Configuring Compile Options

Before chrooting into your new environment, you can configure some global compile options. These are typically set in `/etc/portage/make.conf`.

1.  **Open `make.conf` for editing:**

    ```bash
    nano /mnt/gentoo/etc/portage/make.conf
    ```

2.  **Set `CFLAGS` and `CXXFLAGS`:** These define optimization flags for your C and C++ compilers. A good starting point is to optimize for your specific CPU architecture. You can find your CPU flags by running `cat /proc/cpuinfo` and looking for the `flags` line.

    ```ini
    # Example for a modern Intel/AMD CPU
    CFLAGS="-march=native -O2 -pipe"
    CXXFLAGS="${CFLAGS}"
    ```

3.  **Set `MAKEOPTS`:** This defines the number of parallel jobs `make` will run. A common recommendation is `(number of CPU cores) + 1`.

    ```ini
    MAKEOPTS="-j5" # Example for a quad-core CPU
    ```

4.  **Save and exit** `make.conf`.

With the stage file extracted and basic compile options set, you are now ready to chroot into your new Gentoo environment.




## 6. Installing the Gentoo Base System

Now that the stage file is extracted, it's time to chroot into your new Gentoo environment and begin building the base system.

### Chrooting into the New Environment

Chrooting (change root) allows you to operate within your new Gentoo installation as if it were already the running system. This is a critical step.

1.  **Copy DNS Information:** This ensures that networking continues to work inside the chrooted environment.

    ```bash
    cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
    ```

2.  **Mount Necessary Filesystems:** These pseudo-filesystems provide essential kernel and device information to the chrooted environment.

    ```bash
    mount --types proc /proc /mnt/gentoo/proc
    mount --rbind /sys /mnt/gentoo/sys
    mount --make-rslave /mnt/gentoo/sys
    mount --rbind /dev /mnt/gentoo/dev
    mount --make-rslave /mnt/gentoo/dev
    mount --bind /run /mnt/gentoo/run
    mount --make-slave /mnt/gentoo/run
    ```

    *Note: If you are using a non-Gentoo installation media and encounter issues with `/dev/shm`, you might need to explicitly mount it as a `tmpfs`.*

    ```bash
    test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
    mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
    chmod 1777 /dev/shm /run/shm
    ```

3.  **Enter the Chroot Environment:**

    ```bash
    chroot /mnt/gentoo /bin/bash
    source /etc/profile
    export PS1="(chroot) ${PS1}"
    ```

    From this point forward, all commands you execute will be within your new Gentoo system.

### Configuring Portage

Portage is Gentoo's powerful package management system. You need to configure it to fetch and manage software effectively.

1.  **Install a Gentoo ebuild repository snapshot:** This snapshot contains metadata about all available software packages.

    ```bash
    emerge-webrsync
    ```

2.  **Select Mirrors (Optional but Recommended):** Choosing geographically closer mirrors can significantly speed up downloads. Edit `/etc/portage/make.conf` and add or uncomment the `GENTOO_MIRRORS` variable.

    ```bash
    nano /etc/portage/make.conf
    ```

    Add or modify the line:

    ```ini
    GENTOO_MIRRORS="http://your.preferred.mirror/gentoo"
    ```

    You can find a list of mirrors at [https://www.gentoo.org/downloads/mirrors/](https://www.gentoo.org/downloads/mirrors/).

3.  **Update the Gentoo ebuild repository (Optional):** If you've been in the chroot for a while, you might want to update the repository again.

    ```bash
    emerge --sync
    ```

4.  **Read News Items:** Gentoo news items provide important information about system changes, security advisories, and package updates. Always read them!

    ```bash
    eselect news read
    ```

5.  **Choose the Right Profile:** A profile defines the base system settings, including default USE flags, CFLAGS, and more. List available profiles and select one that suits your needs.

    ```bash
    eselect profile list
    ```

    You'll see a list of profiles, often with numbers. Select the appropriate one:

    ```bash
    eselect profile set <number>
    ```

    For most desktop users, a `default/linux/amd64/17.0/desktop` profile (or similar) is a good choice.

6.  **Configure USE Variables (Optional but Important):** USE flags enable or disable features in packages during compilation. They are crucial for customizing your system. You can set global USE flags in `/etc/portage/make.conf`.

    ```bash
    nano /etc/portage/make.conf
    ```

    Add or modify the `USE` variable. For example, to enable `systemd` and `gnome` support globally:

    ```ini
    USE="systemd gnome"
    ```

    You can also set `CPU_FLAGS_X86` to optimize for your CPU's features:

    ```bash
    # Install cpuinfo2cpuflags
emerge --noreplace app-portage/cpuinfo2cpuflags

# Run it to get recommended CPU_FLAGS_X86
cpuinfo2cpuflags

# Add the output to your make.conf
    ```

7.  **Configure `ACCEPT_LICENSE` (Optional):** If you plan to install software with restrictive licenses, you might need to add them to `ACCEPT_LICENSE` in `make.conf`.

    ```ini
    ACCEPT_LICENSE="@FREE @BINARY-REDISTRIBUTABLE"
    ```

8.  **Update the `@world` set (Optional):** This ensures all installed packages (currently just the stage3) are consistent with your chosen profile and USE flags.

    ```bash
    emerge --ask --update --deep --newuse @world
    ```

### Timezone

Set your system's timezone. First, list available timezones:

```bash
ls /usr/share/zoneinfo
# Then, for example, for America/New_York:
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
```

### Configure Locales

Locales define language, character sets, and other regional settings. 

1.  **Edit `/etc/locale.gen`:** Uncomment the locales you wish to generate (e.g., `en_US.UTF-8 UTF-8`).

    ```bash
nano /etc/locale.gen
    ```

2.  **Generate locales:**

    ```bash
    locale-gen
    ```

3.  **Select your default locale:**

    ```bash
    eselect locale list
    eselect locale set <number>
    ```

4.  **Reload the environment:**

    ```bash
    env-update && source /etc/profile
    ```

With the base system configured, the next step is to compile and configure your Linux kernel.




## 7. Configuring the Linux Kernel

The Linux kernel is the heart of your Gentoo system. You have several options for configuring and compiling it. For most users, using a distribution kernel or manually configuring it are the most common approaches.

### Optional: Installing Firmware and Microcode

Some hardware components require proprietary firmware or microcode to function correctly. It's recommended to install these early.

1.  **Install Linux Firmware:** This package contains firmware for many devices, especially wireless cards and GPUs.

    ```bash
    emerge --ask sys-kernel/linux-firmware
    ```

    *Note: You might need to accept certain licenses for proprietary firmware. If prompted, review the licenses and accept them if you agree.*

2.  **Install Intel Microcode (if applicable):** If you have an Intel CPU, installing microcode updates can improve stability and security.

    ```bash
    emerge --ask sys-firmware/intel-microcode
    ```

    *AMD CPU microcode is typically included in `sys-kernel/linux-firmware`.*

### Kernel Configuration and Compilation

There are two primary ways to configure and compile your kernel: using a distribution kernel (recommended for simplicity) or manual configuration (for advanced users).

#### Option 1: Using a Distribution Kernel (Recommended)

Gentoo provides pre-configured kernel sources that simplify the compilation process. The `gentoo-sources` package is a good starting point.

1.  **Install `gentoo-sources`:**

    ```bash
    emerge --ask sys-kernel/gentoo-sources
    ```

2.  **Install `genkernel` (Optional, but simplifies initial setup):** `genkernel` is a tool that automates much of the kernel configuration and compilation process.

    ```bash
    emerge --ask sys-kernel/genkernel
    ```

3.  **Configure and Compile the Kernel with `genkernel`:**

    ```bash
    genkernel all
    ```

    This command will automatically configure, compile, and install your kernel and initramfs. This process can take a significant amount of time depending on your system's resources.

#### Option 2: Manual Kernel Configuration (Advanced)

Manual configuration gives you granular control over every kernel option. This is powerful but requires a good understanding of your hardware and kernel functionalities.

1.  **Install `gentoo-sources`:**

    ```bash
    emerge --ask sys-kernel/gentoo-sources
    ```

2.  **Navigate to the kernel source directory:**

    ```bash
    cd /usr/src/linux
    ```

3.  **Configure the kernel:**

    ```bash
    make menuconfig
    ```

    This will open a text-based configuration interface. You'll need to enable essential drivers for your hardware (e.g., disk controllers, network cards) and file systems (e.g., Ext4, XFS). Pay close attention to:

    *   `General setup`
    *   `Enable loadable module support`
    *   `Processor type and features`
    *   `Device Drivers` (especially `SCSI device support`, `Network device support`, `Input device support`)
    *   `File systems`

    *Tip: If unsure, you can start with a default configuration by running `make defconfig` before `make menuconfig`.*

4.  **Compile the kernel:**

    ```bash
    make -j$(nproc)
    ```

    This will compile the kernel. The `-j$(nproc)` option uses all available CPU cores for faster compilation.

5.  **Install the kernel modules:**

    ```bash
    make modules_install
    ```

6.  **Install the kernel:**

    ```bash
    make install
    ```

    This will copy the kernel image and System.map to `/boot`.

### Kernel Modules

After compiling the kernel, you might need to load specific modules for your hardware. You can list available modules with `find /lib/modules/<kernel-version> -type f -name '*.ko'`.

To force load a module (e.g., `nvidia`):

```bash
modprobe nvidia
```

To make a module load at boot, add it to `/etc/modules-load.d/<filename>.conf`:

```bash
echo 'nvidia' > /etc/modules-load.d/nvidia.conf
```

With your kernel configured and installed, you're now ready to configure the rest of your Gentoo system.




## 8. Configuring the System

After the kernel is configured, it's time to set up the core system configuration files. This includes `fstab`, network settings, hostname, and root password.

### Filesystem Information (`/etc/fstab`)

The `/etc/fstab` file defines how your system mounts filesystems. It's crucial for ensuring your partitions are mounted correctly at boot.

1.  **Understanding `fstab` entries:** Each line in `fstab` typically has six fields:
    *   **Field 1:** The device to be mounted (e.g., `/dev/sda1`, `UUID=...`, `LABEL=...`). Using `UUID` or `LABEL` is generally recommended for robustness as device names like `/dev/sda1` can change.
    *   **Field 2:** The mount point (where the filesystem will be accessible, e.g., `/`, `/boot`, `/home`).
    *   **Field 3:** The filesystem type (e.g., `ext4`, `xfs`, `swap`).
    *   **Field 4:** Mount options (e.g., `defaults`, `noatime`, `ro`).
    *   **Field 5:** Used by `dump` (usually `0`).
    *   **Field 6:** Used by `fsck` to determine the check order (root filesystem is `1`, others are `2` or `0` for no check).

2.  **Identify your partitions:** Use `blkid` to find the `UUID` or `PARTUUID` of your partitions. This is the most reliable way to identify them in `fstab`.

    ```bash
    blkid
    ```

3.  **Create/Edit `/etc/fstab`:** Open the file with your preferred text editor (e.g., `nano`).

    ```bash
    nano /etc/fstab
    ```

    Here's an example `fstab` configuration. **Adjust this to match your specific partitioning scheme and chosen filesystems.**

    ```ini
    # /etc/fstab: static file system information.
    #
    # <file system> <mount point>   <type>  <options>       <dump>  <pass>

    # For the root partition (replace UUID with your actual root partition's UUID)
    UUID=<your_root_uuid> /               ext4    defaults,noatime 0 1

    # For the boot partition (replace UUID with your actual boot partition's UUID)
    UUID=<your_boot_uuid> /boot           ext4    defaults        0 2

    # For the EFI System Partition (if UEFI, replace UUID)
    UUID=<your_efi_uuid>  /efi            vfat    defaults        0 2

    # For swap (replace UUID with your actual swap partition's UUID)
    UUID=<your_swap_uuid> none            swap    sw              0 0

    # For a separate /home partition (if you created one)
    # UUID=<your_home_uuid> /home           ext4    defaults,noatime 0 2

    # Example for a CD-ROM drive
    /dev/cdrom      /mnt/cdrom      auto    noauto,user,ro  0 0
    ```

    *   **Important:** If you are using UEFI and your partitions conform to the Discoverable Partition Specification, `systemd` might auto-mount them, potentially making `fstab` less critical for those partitions. However, it's still good practice to have a complete `fstab`.

### Networking Information

Configure your system's hostname and network settings.

1.  **Set the Hostname:** Choose a unique name for your system.

    ```bash
    echo "mygentoobox" > /etc/hostname
    ```

2.  **Configure Network (OpenRC):** If you are using OpenRC, you'll typically use `netifrc`.

    *   **Install `dhcpcd` (if not already installed):**

        ```bash
        emerge --ask net-misc/dhcpcd
        ```

    *   **Configure your network interface:** Edit `/etc/conf.d/net`.

        ```bash
        nano /etc/conf.d/net
        ```

        For DHCP, add the following (replace `eth0` with your actual interface name, e.g., `enp0s3`):

        ```ini
        config_eth0="dhcp"
        ```

    *   **Automatically start networking at boot:**

        ```bash
        cd /etc/init.d
        ln -s net.lo net.eth0 # Replace eth0 with your interface
        rc-update add net.eth0 default
        ```

3.  **Configure `/etc/hosts`:** This file maps hostnames to IP addresses. It's good practice to include your hostname here.

    ```bash
    nano /etc/hosts
    ```

    Add entries similar to these, replacing `mygentoobox` with your chosen hostname:

    ```ini
    127.0.0.1       localhost
    ::1             localhost
    127.0.0.1       mygentoobox.localdomain mygentoobox
    ```

### System Information

1.  **Set the Root Password:** This is critical for securing your system.

    ```bash
    passwd
    ```

    Enter and confirm a strong password for the `root` user.

2.  **Add a Regular User (Highly Recommended):** You should not operate as `root` for daily tasks. Create a new user account.

    ```bash
    useradd -m -G users,wheel,audio,video,usb,cdrom,portage,cron -s /bin/bash yourusername
    passwd yourusername
    ```

    *   Replace `yourusername` with your desired username.
    *   The `-G` flag adds the user to various common groups. Adjust as needed.
    *   The `wheel` group is important if you plan to use `sudo`.

3.  **Configure `sudo` (Optional):** If you want your regular user to be able to execute commands as `root` using `sudo`, you need to install it and configure it.

    ```bash
    emerge --ask app-admin/sudo
    ```

    Then, edit the `sudoers` file using `visudo` (always use `visudo` to prevent syntax errors):

    ```bash
    visudo
    ```

    Uncomment the line that allows members of the `wheel` group to use `sudo`:

    ```ini
    %wheel ALL=(ALL) ALL
    ```

4.  **Init and Boot Configuration:**

    *   **OpenRC:** Services are managed with `rc-update`. To enable a service to start at boot, use:

        ```bash
        rc-update add <service_name> default
        ```

    *   **systemd:** Services are managed with `systemctl`. To enable a service to start at boot, use:

        ```bash
        systemctl enable <service_name>
        ```

With these system configurations in place, your Gentoo system is taking shape. The next step is to install essential system tools.




## 9. Installing System Tools

Now that the core system is configured, it's time to install some essential system tools that will make your Gentoo experience more complete and user-friendly.

### System Logger

A system logger is crucial for monitoring your system's health and troubleshooting issues. Gentoo offers several options:

*   **`sysklogd` (app-admin/sysklogd):** A traditional and simple logger, good for beginners.
*   **`syslog-ng` (app-admin/syslog-ng):** An advanced logger with powerful filtering and routing capabilities, but requires more configuration.
*   **`metalog` (app-admin/metalog):** A highly configurable logger.

For most users, `sysklogd` is a good choice to start with.

```bash
emerge --ask app-admin/sysklogd
rc-update add sysklogd default
```

*If you are using `systemd`, `systemd-journald` is built-in and usually sufficient, so you can skip installing a separate system logger.*

### Cron Daemon (Optional but Recommended)

A cron daemon allows you to schedule tasks to run automatically at specified intervals (e.g., daily, weekly, monthly). This is invaluable for system maintenance, backups, and other routine operations.

Popular cron daemons include:

*   **`cronie` (sys-process/cronie):** Based on the original cron, with security enhancements.
*   **`dcron` (sys-process/dcron):** A lightweight and secure option.
*   **`fcron` (sys-process/fcron):** Offers extended capabilities.
*   **`bcron` (sys-process/bcron):** Designed with security in mind, using privilege separation.

`cronie` is a solid choice for most users.

```bash
emerge --ask sys-process/cronie
rc-update add cronie default
```

*If you are using `systemd`, you can use `systemd` timers for scheduled tasks, which are built-in and often eliminate the need for a separate cron daemon.*

### File Indexing (Optional)

To quickly locate files on your system, you can install a file indexing utility like `mlocate`.

```bash
emerge --ask sys-apps/mlocate
```

### Remote Shell Access (Optional)

If you plan to access your Gentoo system remotely via SSH, you'll need to install and configure `openssh`.

```bash
emerge --ask net-misc/openssh
```

*   **OpenRC:**

    ```bash
    rc-update add sshd default
    ```

*   **systemd:**

    ```bash
    systemctl enable sshd
    ```

*   **Important:** By default, `sshd` does not allow `root` login. It's highly recommended to create a non-root user (as described in the 


previous section) and use `sudo` for administrative tasks, or adjust `/etc/ssh/sshd_config` to allow root login (not recommended for security reasons).

### Shell Completion (Optional)

Shell completion can greatly improve your command-line experience by providing tab-completion for commands and arguments. For Bash, you can install `bash-completion`.

```bash
emerge --ask app-shells/bash-completion
```

After installation, you might need to source the completion script in your `~/.bashrc` or `/etc/bash.bashrc`.

### Time Synchronization (Suggested)

Keeping your system clock accurate is important for many reasons, including proper logging and secure communication. `ntp` or `chrony` are common choices.

```bash
emerge --ask net-misc/ntp
```

*   **OpenRC:**

    ```bash
    rc-update add ntpd default
    ```

*   **systemd:**

    ```bash
    systemctl enable ntpd
    ```

### Filesystem Tools

Ensure you have the necessary tools for managing your filesystems. For `ext4`, `xfs`, `btrfs`, etc., you'll typically install the corresponding `xfsprogs`, `btrfs-progs`, etc.

```bash
emerge --ask sys-fs/xfsprogs # For XFS filesystems
emerge --ask sys-fs/e2fsprogs # For Ext2/3/4 filesystems
```

### Networking Tools

While you configured basic networking earlier, you might need additional tools depending on your setup.

*   **DHCP Client:** `dhcpcd` is a common choice.

    ```bash
emerge --ask net-misc/dhcpcd
    ```

*   **Wireless Networking Tools (if applicable):** For wireless connections, you'll need `wpa_supplicant` and potentially `iw`.

    ```bash
emerge --ask net-wireless/wpa_supplicant
emerge --ask net-wireless/iw
    ```

With these essential tools installed, your Gentoo system is becoming more functional. The next crucial step is to install and configure a bootloader to make your system bootable.




## 10. Configuring the Bootloader

The bootloader is the final critical component to install, as it's responsible for initiating the Linux kernel when your system boots up. Without it, your Gentoo system won't be able to start.

For AMD64 systems, the Gentoo Handbook primarily focuses on GRUB, but also mentions `systemd-boot` and `EFI Stub` for UEFI systems.

### Default: GRUB

GRUB (Grand Unified Bootloader) is the most common bootloader for Linux systems, offering broad compatibility with both traditional BIOS and modern UEFI systems.

1.  **Emerge GRUB:**

    *   **For BIOS systems (MBR partition tables):**

        ```bash
        emerge --ask --verbose sys-boot/grub
        ```

    *   **For UEFI systems:** You need to ensure GRUB is built with EFI support. Add `GRUB_PLATFORMS="efi-64"` to your `/etc/portage/make.conf` *before* emerging GRUB.

        ```bash
        echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
        emerge --ask --verbose sys-boot/grub
        ```

        If you already emerged GRUB without this flag, you can add the line to `make.conf` and then re-emerge:

        ```bash
        emerge --ask --update --newuse --verbose sys-boot/grub
        ```

2.  **Install GRUB to Disk:** This step physically writes the bootloader to your disk.

    *   **For DOS/Legacy BIOS systems:** Replace `/dev/sda` with your boot disk.

        ```bash
        grub-install /dev/sda
        ```

    *   **For UEFI systems:** Ensure your EFI System Partition (ESP) is mounted (e.g., at `/efi`).

        ```bash
        grub-install --efi-directory=/efi
        ```

        You should see a success message like `Installation finished. No error reported.`

3.  **Configure GRUB:** Generate the GRUB configuration file. This file tells GRUB where your kernel is and how to boot it.

    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

    Review the output to ensure your kernel and initramfs (if you generated one) are detected correctly.

### Alternative 1: `systemd-boot` (for UEFI systems)

`systemd-boot` is a simple UEFI boot manager that reads configuration files and boots UEFI executables. It's often preferred for its simplicity on UEFI-only systems.

1.  **Emerge `systemd-boot`:**

    *   **On OpenRC systems:**

        ```bash
        echo 'sys-apps/systemd boot kernel-install' >> /etc/portage/package.use/systemd-boot
        echo 'sys-kernel/installkernel systemd systemd-boot' >> /etc/portage/package.use/systemd-boot
        emerge --ask sys-apps/systemd-utils sys-kernel/installkernel
        ```

    *   **On systemd systems:**

        ```bash
        echo 'sys-apps/systemd boot' >> /etc/portage/package.use/systemd
        echo 'sys-kernel/installkernel systemd-boot' >> /etc/portage/package.use/systemd
        emerge --ask sys-apps/systemd sys-kernel/installkernel
        ```

2.  **Install `systemd-boot`:**

    ```bash
    bootctl install
    ```

3.  **Configure `systemd-boot`:** Create boot entries in `/boot/loader/entries/` (e.g., `gentoo.conf`).

    ```ini
    # /boot/loader/entries/gentoo.conf
    title   Gentoo Linux
    linux   /vmlinuz-<kernel-version>
    initrd  /initramfs-<kernel-version>.img # If you generated an initramfs
    options root=UUID=<your_root_uuid> rw
    ```

    Replace `<kernel-version>` and `<your_root_uuid>` with your actual values.

### Alternative 2: EFI Stub (for UEFI systems)

EFI Stub allows the kernel to be directly bootable by the UEFI firmware, bypassing the need for a separate bootloader like GRUB or `systemd-boot`. This is the most minimalist approach.

1.  **Ensure Kernel is Built with EFI Stub Support:** When configuring your kernel (manual configuration), ensure `CONFIG_EFI_STUB=y` is enabled.

2.  **Copy Kernel to ESP:** Copy your kernel image to the EFI System Partition (ESP), typically mounted at `/efi`.

    ```bash
    cp /boot/vmlinuz-<kernel-version> /efi/EFI/Gentoo/gentoo-kernel.efi
    ```

3.  **Create a Boot Entry:** Use `efibootmgr` to create a new UEFI boot entry.

    ```bash
    efibootmgr --create --disk /dev/sda --part 1 --label "Gentoo Linux" --loader /EFI/Gentoo/gentoo-kernel.efi --unicode 'root=UUID=<your_root_uuid> rw'
    ```

    Replace `/dev/sda` and `--part 1` with your ESP's disk and partition number, and `<your_root_uuid>` with your root partition's UUID.

### Rebooting the System

Once the bootloader is configured, you are ready to reboot into your new Gentoo system!

1.  **Exit the chroot environment:**

    ```bash
    exit
    ```

2.  **Unmount all partitions:**

    ```bash
    umount -l /mnt/gentoo/dev{/shm,/pts,}
    umount -R /mnt/gentoo
    ```

3.  **Reboot:**

    ```bash
    reboot
    ```

Congratulations! You have successfully installed Gentoo Linux. The next section will cover some final steps and post-installation tasks.




## 11. Finalizing the Installation and Post-Installation Tasks

Congratulations! You've successfully installed Gentoo Linux. Now, let's cover some essential post-installation steps to get your system fully operational and user-friendly.

### Updating Your System

Regularly updating your system is crucial for security, stability, and access to the latest software. Gentoo's Portage system makes this straightforward.

```bash
emerge --sync         # Synchronize the Portage tree with the latest ebuilds
emerge --ask --update --deep --newuse @world # Update all installed packages
```

*   `--update`: Updates packages to the latest version.
*   `--deep`: Ensures all dependencies are updated.
*   `--newuse`: Reinstalls packages if their USE flags have changed.
*   `@world`: Refers to all packages explicitly installed by you.

### Cleaning Up Old Packages

After updates, you might have orphaned packages (dependencies no longer needed) or old distfiles. Clean them up to save disk space.

```bash
emerge --ask --depclean # Remove orphaned packages
revdep-rebuild          # Rebuild packages that might have broken dependencies
eclean distfiles        # Clean up old source tarballs
eclean packages         # Clean up old binary packages
```

### Kernel Updates

When a new kernel version is available, you'll need to recompile and install it. The process is similar to the initial kernel compilation.

1.  **Update `gentoo-sources`:**

    ```bash
    emerge --ask sys-kernel/gentoo-sources
    ```

2.  **Recompile and Install Kernel:**

    *   **If using `genkernel`:**

        ```bash
        genkernel all
        ```

    *   **If manual configuration:**

        ```bash
        cd /usr/src/linux
        make oldconfig # Use your old .config, prompts for new options
        # OR make menuconfig for full configuration
        make -j$(nproc)
        make modules_install
        make install
        ```

3.  **Update Bootloader Configuration:**

    *   **GRUB:**

        ```bash
        grub-mkconfig -o /boot/grub/grub.cfg
        ```

    *   **`systemd-boot`:** Update your `/boot/loader/entries/*.conf` files with the new kernel version.

### Installing a Desktop Environment (Optional)

If you want a graphical user interface, you'll need to install a display server (Xorg) and a desktop environment (e.g., KDE Plasma, GNOME, Xfce).

1.  **Install Xorg:**

    ```bash
    emerge --ask x11-base/xorg-server
    ```

2.  **Install a Desktop Environment (Example: Xfce):**

    ```bash
    emerge --ask xfce-base/xfce4-meta
    ```

    *Note: Desktop environments are large and will take a long time to compile.*

3.  **Configure Display Manager (e.g., LightDM, GDM, SDDM):**

    ```bash
    emerge --ask x11-misc/lightdm
    rc-update add lightdm default # For OpenRC
    # systemctl enable lightdm     # For systemd
    ```

### Sound Configuration

Install ALSA (Advanced Linux Sound Architecture) for sound support.

```bash
emerge --ask media-libs/alsa-lib media-plugins/alsa-utils
```

Add your user to the `audio` group:

```bash
usermod -aG audio yourusername
```

### Browser and Other Applications

Install your favorite web browser and other applications.

```bash
emerge --ask www-client/firefox
emerge --ask app-office/libreoffice
```
END
---


