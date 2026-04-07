# The Linux File System: Structure, Purpose & Practice

> A complete guide to the Linux File Hierarchy Standard (FHS) — what lives where, why it's organized that way, and how to work with it effectively.

---

## Table of Contents

- [The Linux File System: Structure, Purpose \& Practice](#the-linux-file-system-structure-purpose--practice)
  - [Table of Contents](#table-of-contents)
  - [1. Why Linux Has This Structure](#1-why-linux-has-this-structure)
    - [A Brief History: DOS, Windows, and UNIX](#a-brief-history-dos-windows-and-unix)
    - [The File Hierarchy Standard (FHS)](#the-file-hierarchy-standard-fhs)
    - [Two Key Axes](#two-key-axes)
  - [2. The Root: `/`](#2-the-root-)
  - [3. User Space](#3-user-space)
    - [`/home` — Your Personal Space](#home--your-personal-space)
    - [`/root` — The Superuser's Home](#root--the-superusers-home)
  - [4. Executables \& Commands](#4-executables--commands)
    - [`/bin` — Essential User Binaries](#bin--essential-user-binaries)
    - [`/sbin` — System Administration Binaries](#sbin--system-administration-binaries)
    - [`/usr` — User Application Space](#usr--user-application-space)
    - [`/opt` — Optional / Third-Party Software](#opt--optional--third-party-software)
  - [5. Libraries](#5-libraries)
    - [`/lib`, `/lib32`, `/lib64` — Shared Libraries](#lib-lib32-lib64--shared-libraries)
  - [6. Boot \& Kernel](#6-boot--kernel)
    - [`/boot` — Boot Loader Files](#boot--boot-loader-files)
    - [`/sys` — Kernel Interface](#sys--kernel-interface)
    - [`/proc` — Process \& Kernel Information](#proc--process--kernel-information)
  - [7. Hardware \& Devices](#7-hardware--devices)
    - [`/dev` — Device Files](#dev--device-files)
  - [8. Configuration](#8-configuration)
    - [`/etc` — System-Wide Configuration](#etc--system-wide-configuration)
  - [9. Variable \& Runtime Data](#9-variable--runtime-data)
    - [`/var` — Variable Data](#var--variable-data)
    - [`/run` — Runtime Data (tmpfs)](#run--runtime-data-tmpfs)
    - [`/tmp` — Temporary Files](#tmp--temporary-files)
  - [10. Storage \& Mounts](#10-storage--mounts)
    - [`/media` — Removable Media (Auto-mounted)](#media--removable-media-auto-mounted)
    - [`/mnt` — Manual Mount Point](#mnt--manual-mount-point)
  - [11. Server Data](#11-server-data)
    - [`/srv` — Service Data](#srv--service-data)
  - [12. Distro-Specific Directories](#12-distro-specific-directories)
    - [`/snap` — Snap Packages (Ubuntu)](#snap--snap-packages-ubuntu)
  - [13. Quick Reference Summary](#13-quick-reference-summary)

---

## 1. Why Linux Has This Structure

### A Brief History: DOS, Windows, and UNIX

To understand how and why Linux organizes its files in a particular way, we will justapose it with Windows.

**The DOS/Windows lineage** begins with MS-DOS (Microsoft Disk Operating System), a command-line-only environment that predates graphical interfaces. In DOS, everything revolves around *drives* identified by letters. The letters `A:` and `B:` were reserved for floppy disk drives which were physically removable floppy disks. When hard drives arrived, `C:` became the conventional name for the primary internal drive, and additional drives or partitions followed alphabetically (`D:`, `E:`, and so on). Windows started as a graphical desktop environment (a GUI) that ran *on top of* MS-DOS. Windows NT onward, Microsoft gradually replaced DOS with a fully independent kernel (removing any DOS dependency towards independent boot), but the drive-letter convention and the directory structure it inherited (like `C:\Program Files\`, `C:\Windows\System32\`) remain to this day.

**The UNIX/Linux lineage** is fundamentally different. UNIX designed its file system around a single *tree* rooted at `/` (called "the root"). Everything including hard drives, removable media, network shares, even hardware devices appear in a well-principled region on this tree. There are no drive letters; instead, a disk partition or USB stick gets "mounted" at a directory within the tree, and from that point on, it's accessed as if it were just a subdirectory. This unified namespace is elegant and flexible for *multi-user, networked environments*.

Linux follows UNIX tradition:

- Paths use **forward slashes**: `/home/shaswat/documents/thesis.pdf`
- **Filenames are case-sensitive**: `File.txt`, `file.txt`, and `FILE.TXT` are three distinct files
- **Hidden files start with a dot**: `.bashrc`, `.config/` and any file or directory beginning with `.` is hidden from normal directory listings (use `ls -a` to reveal them)

> **What is POSIX?** POSIX (Portable Operating System Interface) is a family of standards defined by the IEEE that specifies how operating systems should behave. what system calls exist, how the shell works, what utilities are available, and how the file system should be structured. Being "POSIX-compliant" means a system follows these rules, making software portable across UNIX-like systems. Linux is largely POSIX-compliant, which is why shell scripts and programs written on macOS often work on Linux with little or no modification.

---

### The File Hierarchy Standard (FHS)

The **File Hierarchy Standard (FHS)** is guiding standard laid down and maintained by the Linux Foundation defining the strcuture, function, principles and purpose for the organization and membership of the file hierarchy with conventions.

The latest version, FHS 3.0, is available at: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html

---

### Two Key Axes

The FHS organizes the file system along two independent axes:

**Axis 1: Shareable vs. Non-Shareable**

A *shareable* file is one that can safely be stored on one machine and accessed by another over a network. Your home directory contents (`/home`) are shareable — a user's files are the same whether accessed from their laptop or from a server. In contrast, *device lock files* (files that record "this device is currently in use") are non-shareable, because they reflect the local state of a specific machine's hardware and would be meaningless or harmful if another machine tried to read them.

**Axis 2: Static vs. Variable**

A *static* file is one that does not change unless a system administrator explicitly modifies it. Files like binaries, libraries, documentation. A *variable* file changes during normal operation, such as log files grow, mail spools fill up, databases update. This distinction matters for two practical reasons: (1) static files can be on a read-only filesystem for security, and (2) variable files need more frequent backup.

These two axes carve the filesystem into four quadrants:

| | **Static** | **Variable** |
|---|---|---|
| **Shareable** | `/usr`, `/opt` | `/var/mail`, `/var/spool` |
| **Non-Shareable** | `/etc`, `/boot` | `/var/run`, `/var/lock` |

> **Why did `/var` get created?** Originally, `/usr` held both binaries and the log/spool data those programs produced. As systems scaled, keeping static and variable data together became a problem — you couldn't mount `/usr` read-only if programs were constantly writing logs into it. The solution was to create `/var` and migrate all variable data there, leaving `/usr` purely static and safely mountable read-only.

---

## 2. The Root: `/`

The root directory is the top of the entire file system tree. Every path on the system begins here. There is only one root, and it lives on the root partition — the partition that the kernel mounts first when booting.

You almost never create files directly in `/`. It contains only the top-level directories described in this guide. If you run `ls /`, you'll see the complete inventory of your system's organizational structure.

```bash
# See everything at the root level
ls /

# See it in long format, including hidden entries
ls -la /
```

---

## 3. User Space

### `/home` — Your Personal Space

Every user on the system gets a dedicated subdirectory inside `/home`. For a user named `shaswat`, this is `/home/shaswat`. This is where all personal files, documents, downloads, and user-specific application configuration live. When you open a terminal, you start here by default.

**What lives inside your home directory:**

Your home directory contains both *visible* files and folders (Documents, Downloads, Desktop, Pictures, etc.) and *hidden* configuration files and directories. The hidden ones are identified by a leading dot and are created automatically by applications the first time they run, and store per-user settings so that configuration is isolated between users on the same machine.

The three most important hidden directories follow the **XDG Base Directory Specification**, a standard that organizes where applications store their data:

- **`.config/`** — Application configuration files. For example, `.config/nvim/init.lua` (Neovim config), `.config/Code/` (VS Code settings). This is the first place to look when you want to customize how an application behaves.
- **`.local/`** — User-specific data and binaries. `.local/share/` holds application data (like installed fonts or desktop entries), and `.local/bin/` is a conventional place to install personal scripts without needing root.
- **`.cache/`** — Cached data that applications generate to speed up future runs. Thumbnails, compiled bytecode, package indexes. This directory is safe to delete — everything here can be regenerated, though applications may be slower the first time they run again.

A few common top-level dotfiles you'll encounter:

- **`.bashrc`** — Executed every time you open a new interactive Bash shell. Where you define aliases, environment variables, and shell behavior.
- **`.bash_profile`** or **`.profile`** — Executed only on login. Often sources `.bashrc` and sets `PATH`.
- **`.ssh/`** — SSH keys and known hosts. Keep the permissions tight: `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/id_rsa`.

**Common recipes:**

```bash
# Navigate to your home directory from anywhere
cd ~
# or simply
cd

# List all files including hidden ones
ls -la ~

# Find all dotfiles/dotdirs at the top level of home
ls -d ~/.*

# Back up your entire home directory configuration to a tarball
tar -czf ~/backup_config_$(date +%Y%m%d).tar.gz ~/.config ~/.local ~/.bashrc ~/.ssh

# See how much disk space your home directory is using
du -sh ~

# Check what's consuming the most space inside home
du -sh ~/* | sort -rh | head -20

# Open the config for an application (example: git)
cat ~/.gitconfig

# Add a personal script to your PATH permanently
mkdir -p ~/.local/bin
cp my_script.sh ~/.local/bin/my_script
chmod +x ~/.local/bin/my_script
# Add to ~/.bashrc: export PATH="$HOME/.local/bin:$PATH"
```

---

### `/root` — The Superuser's Home

`/root` is the home directory of the `root` user or the system's superuser (administrator). It exists at `/root` rather than `/home/root` for a deliberate reason: the `/home` directory might be on a separate disk partition. If that partition fails to mount during boot, the root user would be locked out of their own home directory, which would be catastrophic during system recovery. By keeping `/root` on the root partition itself, the superuser always has access.

Normal users cannot read or write to `/root`. It's protected by permissions (mode `0700` by default — read, write, execute for owner only).

```bash
# Switch to the root user (if you have sudo rights)
sudo -i

# Run a single command as root without switching sessions
sudo some_command

# Check who you currently are
whoami

# Check the effective user ID (0 = root)
id
```

> **A note on security:** Even if you have `sudo` access, the best practice is to avoid working as root directly. Use `sudo` for individual commands that require it. This limits the blast radius of mistakes. A mistyped `rm -rf` as root can destroy the entire system, while the same mistake as a normal user only destroys your home directory.

---

## 4. Executables & Commands

### `/bin` — Essential User Binaries

> **What is a binary?** "Binary" is shorthand for a compiled, executable program, as opposed to a script, which is plain text. When you write a C program and compile it, the output is a binary: a file containing machine code that the processor can execute directly.

`/bin` contains the essential command-line tools that every user needs for the system to be functional. These commands must be available even in single-user mode (a minimal recovery environment) or before other partitions are mounted. Examples include:

| Command | What it does |
|---|---|
| `ls` | List directory contents |
| `cat` | Concatenate and print files |
| `cp`, `mv`, `rm` | Copy, move, delete files |
| `grep` | Search text using patterns |
| `echo` | Print text to the terminal |
| `bash`, `sh` | The shell itself |
| `mkdir`, `rmdir` | Create and remove directories |
| `chmod`, `chown` | Change file permissions and ownership |
| `ps` | List running processes |
| `kill` | Send signals to processes |

On modern Linux systems (Ubuntu 20.04+, Fedora, Arch), `/bin` is often a **symbolic link** to `/usr/bin` as part of the "UsrMerge" initiative, which consolidates binaries in one place for simplicity. You can check: `ls -la /bin` — if it shows `bin -> usr/bin`, your system has merged them.

```bash
# Find where a command lives
which ls          # → /usr/bin/ls (or /bin/ls)
type grep         # shows whether it's a binary, shell function, or alias

# List everything in /bin sorted by size (useful for auditing)
ls -lS /bin | head -20

# Find all executable files in /bin
find /bin -type f -executable

# Check what a binary does (if it has a manual page)
man cat
```

---

### `/sbin` — System Administration Binaries

`/sbin` holds binaries intended for system administration tasks. These are commands that typically require root privileges and are not needed for routine user activity. Think of it as the administrator's toolkit.

Examples:

| Command | What it does |
|---|---|
| `fdisk` / `parted` | Manage disk partitions |
| `mkfs` | Format a disk with a filesystem |
| `fsck` | Check and repair a filesystem |
| `iptables` / `ip` | Configure network interfaces and firewall rules |
| `mount` / `umount` | Attach and detach filesystems |
| `reboot`, `shutdown` | Control system power state |
| `adduser`, `userdel` | Manage user accounts |
| `sshd` | The SSH server daemon |

Like `/bin`, on modern systems `/sbin` is often a symlink to `/usr/sbin`.

```bash
# List all network interfaces (requires /sbin/ip)
ip addr show

# Check disk partition table
sudo fdisk -l

# Check filesystem integrity (run on unmounted partition)
sudo fsck /dev/sdb1

# Add a new user to the system
sudo adduser newusername

# Shut down the system gracefully in 5 minutes with a message
sudo shutdown -h +5 "System going down for maintenance"

# Cancel a pending shutdown
sudo shutdown -c
```

---

### `/usr` — User Application Space

`/usr` is the largest directory on most Linux systems and the most layered in terms of structure. The name originally stood for "Unix System Resources" (not "user" as one might guess). It holds *non-essential* software; everything the system needs to run (kernel, shell, basic utilities) is in `/bin`, `/sbin`, and `/lib`. Everything else, the applications you actually use, lives in `/usr`.

The key subdirectories within `/usr` are:

**`/usr/bin`** — Binaries for user-installed applications that don't need to be available in single-user mode. This is where most of the programs you interact with daily live: `python3`, `git`, `vim`, `gcc`, `curl`, `wget`, `ssh`, `ffmpeg`, etc.

**`/usr/sbin`** — System administration binaries that aren't needed in single-user mode recovery. Many network services and daemons are here.

**`/usr/lib`** and **`/usr/lib64`** — Libraries (see the dedicated section below) required by programs in `/usr/bin` and `/usr/sbin`.

**`/usr/local/`** — This subtree mirrors the structure of `/usr` (`/usr/local/bin`, `/usr/local/lib`, `/usr/local/etc`) and is specifically reserved for software that the *system administrator* has installed manually — not via the distribution's package manager. If you compile something from source and run `make install`, it typically installs here. The separation means a distribution upgrade can safely overwrite `/usr/bin` and `/usr/lib` without touching your manually installed software in `/usr/local/`.

**`/usr/share/`** — Architecture-independent data that can theoretically be shared across machines: man pages (`/usr/share/man/`), documentation (`/usr/share/doc/`), icons, fonts, locale/internationalization data, and desktop entries for applications.

**`/usr/include/`** — C header files (`.h` files) needed when compiling programs against system libraries. If you're building software from source that links against, say, OpenSSL, the compiler looks here.

```bash
# See how large /usr is
du -sh /usr

# Find all Python 3 executables
find /usr/bin -name "python*"

# Check what package owns a file (Debian/Ubuntu)
dpkg -S /usr/bin/git

# Check what package owns a file (RHEL/Fedora)
rpm -qf /usr/bin/git

# List all files installed by a package (Debian/Ubuntu)
dpkg -L git

# See where manually compiled software ends up
ls /usr/local/bin

# Read the man page for a command (stored in /usr/share/man)
man ssh

# Find all documentation for the git package
ls /usr/share/doc/git/
```

---

### `/opt` — Optional / Third-Party Software

`/opt` is intended for self-contained, optional software packages that are not managed by the system's package manager. A package that installs to `/opt` typically puts everything it needs — binaries, libraries, configuration, data — inside its own subdirectory (e.g., `/opt/google/chrome/`, `/opt/pycharm/`, `/opt/ibm/`). This self-containment means uninstalling it is as simple as deleting the directory.

The distinction between `/usr/local` and `/opt` is subtle but meaningful: `/usr/local` follows the standard FHS tree structure (binaries go to `/usr/local/bin`, libs to `/usr/local/lib`, etc.), while `/opt` is for monolithic application bundles that don't wish to be split across the tree.

Common software you'll find here: JetBrains IDEs, Google Chrome, some enterprise software, manually installed language runtimes.

```bash
# List all third-party software installed in /opt
ls /opt

# Add a binary from /opt to your PATH
export PATH="/opt/pycharm/bin:$PATH"
# Add this to ~/.bashrc to make it permanent

# Create a symlink from /opt binary to /usr/local/bin so it's always in PATH
sudo ln -s /opt/some-tool/bin/some-tool /usr/local/bin/some-tool

# Check how much space an opt package is using
du -sh /opt/google/

# Install a tool manually to /opt (example pattern)
sudo mkdir -p /opt/my-tool
sudo tar -xzf my-tool-v1.0.tar.gz -C /opt/my-tool --strip-components=1
```

---

## 5. Libraries

### `/lib`, `/lib32`, `/lib64` — Shared Libraries

> **What are shared libraries? (And what are DLLs?)**
>
> When a program is compiled, it uses code from external libraries as reusable collections of functions. There are two ways to include that code: (1) *static linking*, where the library code is copied directly into the executable at compile time, making the binary self-contained but large; and (2) *dynamic linking*, where the binary is compiled with only a reference to the library, and the actual code is loaded at runtime from a shared file.
>
> On Linux, these shared files are called **shared objects** and have the extension `.so` (e.g., `libc.so.6`). On Windows, the equivalent is a **DLL** (Dynamic Link Library), files with the `.dll` extension. The concept is identical. A single file containing compiled library code that multiple programs can share simultaneously in memory, reducing both disk usage and RAM consumption.
>
> The advantage is efficiency: if 20 programs all use the C standard library, they share one copy of `libc.so` in memory rather than each embedding their own. The trade-off is the "DLL hell" problem — if a library is updated in a way that breaks its interface, it can break all programs that depend on it simultaneously.

`/lib` contains the essential shared libraries required by the binaries in `/bin` and `/sbin`. These must be present at boot before `/usr` is mounted. The most fundamental ones are the C standard library (`libc.so`) and the dynamic linker/loader itself (`ld-linux.so`).

`/lib32` and `/lib64` exist on 64-bit systems to separately store 32-bit and 64-bit versions of libraries, allowing a 64-bit system to run 32-bit executables (important for legacy software compatibility).

```bash
# List essential system libraries
ls /lib/x86_64-linux-gnu/ | head -20

# Find which library a binary depends on
ldd /bin/ls

# Find the library providing a specific symbol (function)
# (useful when a program fails to load a library)
ldconfig -p | grep libssl

# Check the dynamic linker cache
ldconfig -v 2>/dev/null | grep libz

# Update the library cache after installing a new library
sudo ldconfig
```

---

## 6. Boot & Kernel

### `/boot` — Boot Loader Files

`/boot` contains everything the system needs to start up before the kernel has loaded and before the full filesystem is available. This includes:

- **The kernel image** itself: typically named `vmlinuz-<version>` (e.g., `vmlinuz-6.5.0-35-generic`). "vmlinuz" means "virtual memory Linux, compressed."
- **The initial RAM disk**: `initrd.img-<version>` or `initramfs-<version>`. This is a small, temporary root filesystem loaded into RAM at boot. It contains drivers and tools needed to mount the real root filesystem (e.g., disk encryption tools, RAID drivers). Once the real root is mounted, it hands off control.
- **GRUB configuration**: `grub/grub.cfg` — the configuration file for GRUB (Grand Unified Bootloader), the most common Linux bootloader. GRUB is what shows you the OS selection menu if you dual-boot.
- **UEFI files**: On modern systems with UEFI firmware (the replacement for the old BIOS), the actual boot files live in the **EFI System Partition** (ESP), typically mounted at `/boot/efi`.

> **What happens at boot?** The sequence is: Power on → UEFI/BIOS firmware initializes hardware → GRUB is loaded from the boot partition → GRUB loads the kernel (`vmlinuz`) and initial RAM disk (`initramfs`) → the kernel initializes, mounts `initramfs` as a temporary root, loads necessary drivers → the kernel mounts the real root filesystem → `systemd` (PID 1) starts and brings up all system services.

You almost never need to touch `/boot` manually. The package manager handles kernel updates automatically.

```bash
# List all installed kernel versions
ls /boot/vmlinuz*

# Check which kernel is currently running
uname -r

# Check available disk space in /boot (can fill up if old kernels aren't purged)
df -h /boot

# On Ubuntu: remove old kernels to free up /boot space
sudo apt autoremove --purge

# View GRUB configuration
cat /boot/grub/grub.cfg

# Update GRUB after making changes to /etc/default/grub
sudo update-grub
```

---

### `/sys` — Kernel Interface

`/sys` (the "sysfs" filesystem) provides a structured window into the kernel's internal state and the hardware devices it knows about. It is not stored on disk — it is generated dynamically by the kernel in memory. Every entry in `/sys` is either something you can read (to query kernel state) or write (to change kernel behavior) in real time.

This is primarily used by device drivers and system management tools, not by end users directly. But knowing it exists lets you do powerful low-level inspection.

```bash
# Check the current CPU scaling governor (performance, powersave, etc.)
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Set all CPUs to performance mode (disables power saving)
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Check battery level (on a laptop)
cat /sys/class/power_supply/BAT0/capacity

# Check battery status (Charging, Discharging, Full)
cat /sys/class/power_supply/BAT0/status

# List all PCI devices recognized by the kernel
ls /sys/bus/pci/devices/

# Check if a network interface is up
cat /sys/class/net/eth0/operstate

# Trigger a USB device rescan
echo 1 | sudo tee /sys/bus/usb/devices/usb1/authorized
```

---

### `/proc` — Process & Kernel Information

`/proc` (the "procfs" filesystem) is a virtual filesystem, like `/sys`, that exposes information about running processes and the kernel. Each running process gets a directory named after its **PID** (Process ID). It is a unique natural number assigned by the kernel to identify it. Inside that directory are files that describe everything about the process: memory usage, open file descriptors, the command that launched it, environment variables, and more.

Beyond processes, `/proc` also exposes kernel-wide information.

```bash
# Get the PID of a running process
pgrep firefox

# Inspect a specific process (replace 1234 with actual PID)
ls /proc/1234/            # all available info
cat /proc/1234/status     # human-readable status (memory, threads, state)
cat /proc/1234/cmdline    # command line that launched it
ls -la /proc/1234/fd/     # file descriptors (open files, sockets)

# CPU information (model, cores, cache)
cat /proc/cpuinfo

# Memory information (total, free, cached, swap)
cat /proc/meminfo

# Kernel version and build info
cat /proc/version

# All kernel parameters (tunable settings)
cat /proc/sys/vm/swappiness      # tendency to use swap (0-100)
cat /proc/sys/net/ipv4/ip_forward  # whether to route packets

# Change a kernel parameter at runtime (not persistent across reboot)
echo 10 | sudo tee /proc/sys/vm/swappiness

# See all mounted filesystems
cat /proc/mounts

# See all open network connections
cat /proc/net/tcp
```

> **`/proc` vs `/sys`:** `/proc` is older and slightly messier — it grew organically and contains a mix of process info and kernel info. `/sys` was introduced later to provide a cleaner, more structured interface specifically for hardware and device information. Modern tools prefer `/sys`, but `/proc` remains important for process introspection and many kernel-wide parameters.

---

## 7. Hardware & Devices

### `/dev` — Device Files

One of UNIX's most elegant ideas is that *everything is a file*, including hardware devices. `/dev` contains **device files** which are special files that represent physical or virtual hardware. When a program reads from or writes to one of these files, the kernel translates that operation into the appropriate hardware interaction.

Device files come in two types:

- **Block devices** (prefixed `b` in `ls -l`): Read/write data in fixed-size blocks. Disk drives are block devices. E.g., `/dev/sda` (first SATA drive), `/dev/nvme0n1` (NVMe SSD), `/dev/sdb1` (second drive, first partition).
- **Character devices** (prefixed `c`): Read/write data as a stream, byte by byte. Keyboards, mice, terminals, and webcams are character devices.

Important entries in `/dev`:

| Device File | What it represents |
|---|---|
| `/dev/sda`, `/dev/sdb` | Physical disks (SATA/SCSI) |
| `/dev/sda1`, `/dev/sda2` | Partitions on the first disk |
| `/dev/nvme0n1` | First NVMe drive |
| `/dev/nvme0n1p1` | First partition on that NVMe |
| `/dev/tty` | The current terminal |
| `/dev/ttyS0` | First serial port (COM1) |
| `/dev/null` | The "black hole" — anything written here is discarded |
| `/dev/zero` | Infinite source of zero bytes |
| `/dev/random`, `/dev/urandom` | Cryptographically random bytes |
| `/dev/stdin`, `/dev/stdout`, `/dev/stderr` | Standard streams |
| `/dev/video0` | First webcam |
| `/dev/input/event*` | Raw input events from keyboard/mouse |
| `/dev/loop*` | Loop devices (used to mount disk image files) |

```bash
# List all disk devices and their partitions
lsblk

# Detailed disk and partition info
sudo fdisk -l /dev/sda

# Write zeros to a file (create a 1GB empty file)
dd if=/dev/zero of=~/empty_file.img bs=1M count=1024

# Generate random data (useful for testing, seeding)
dd if=/dev/urandom of=~/random.bin bs=1M count=1

# Mount a disk image (.iso or .img) as if it were a drive
sudo mount -o loop ~/disk.img /mnt/image

# Monitor raw keyboard input events (press keys to see events)
sudo cat /dev/input/event0

# Check if a webcam is detected
ls /dev/video*

# Discard output from a noisy command (send to /dev/null)
some_verbose_command > /dev/null 2>&1

# Check disk health (SMART data)
sudo smartctl -a /dev/sda
```

---

## 8. Configuration

### `/etc` — System-Wide Configuration

`/etc` is the nerve center for system-wide configuration. The name is a remnant from early UNIX and it literally meant "et cetera," a catch-all for files that didn't fit elsewhere. But it has long since become the dedicated home for editable configuration files. The key word is *system-wide*: files here affect all users and all services. User-specific configuration lives in `~/.config/`.

`/etc` contains plain-text files and directories, which makes it possible to manage configuration with standard text editors and version control (many administrators track `/etc` in a Git repository).

Notable files and directories:

| Path | Purpose |
|---|---|
| `/etc/passwd` | User account database (username, UID, home dir, default shell) |
| `/etc/shadow` | Encrypted password hashes (root-readable only) |
| `/etc/group` | Group definitions and memberships |
| `/etc/sudoers` | Who is allowed to use `sudo` and with what privileges |
| `/etc/hostname` | The machine's hostname |
| `/etc/hosts` | Static hostname-to-IP mappings (consulted before DNS) |
| `/etc/resolv.conf` | DNS server configuration |
| `/etc/fstab` | Filesystem mount table (what gets mounted at boot and where) |
| `/etc/crontab` | System-wide scheduled tasks (cron jobs) |
| `/etc/cron.d/` | Additional cron job drop-in files |
| `/etc/apt/` | APT package manager config (Ubuntu/Debian): sources.list, preferences |
| `/etc/ssl/` | SSL/TLS certificates and keys |
| `/etc/ssh/sshd_config` | SSH server configuration |
| `/etc/systemd/` | Systemd service unit files and configuration |
| `/etc/environment` | System-wide environment variables (PATH, etc.) |
| `/etc/profile` | Shell configuration for all login shells |
| `/etc/profile.d/` | Drop-in scripts sourced by `/etc/profile` |
| `/etc/network/` or `/etc/netplan/` | Network interface configuration |

```bash
# View all users on the system
cat /etc/passwd

# Edit the sudoers file safely (validates syntax before saving)
sudo visudo

# Add a new entry to /etc/hosts (e.g., block a domain or alias a server)
echo "127.0.0.1   ads.example.com" | sudo tee -a /etc/hosts

# View all package repository sources (Ubuntu/Debian)
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# View and edit SSH server settings (e.g., disable password auth)
sudo vim /etc/ssh/sshd_config
# After editing, reload the service:
sudo systemctl reload sshd

# Add a system-wide environment variable
echo 'export MY_VAR="value"' | sudo tee /etc/profile.d/myvars.sh

# View what gets auto-mounted at boot
cat /etc/fstab

# Add a disk to /etc/fstab for auto-mount at boot
# (get UUID first with: sudo blkid /dev/sdb1)
echo "UUID=xxxx-xxxx  /mnt/data  ext4  defaults  0  2" | sudo tee -a /etc/fstab

# List all active cron jobs (system-wide)
ls /etc/cron.d/
crontab -l          # your own user's cron jobs
sudo crontab -l     # root's cron jobs

# View SSL certificates
ls /etc/ssl/certs/
```

> **Best practice:** Before editing any file in `/etc`, make a backup: `sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`. Track `/etc` in version control with a tool like `etckeeper` to maintain an audit trail of every change.

---

## 9. Variable & Runtime Data

### `/var` — Variable Data

`/var` holds files that are expected to grow and change continuously during normal system operation. It was split from `/usr` specifically so that `/usr` could be mounted read-only. The key philosophy: if a file is written to by running services (not by administrators), it belongs in `/var`.

Important subdirectories:

| Path | Contents |
|---|---|
| `/var/log/` | System and application log files |
| `/var/lib/` | Persistent application state (databases, package manager state, Docker images) |
| `/var/spool/` | Queued data awaiting processing (print jobs, mail queues, cron jobs) |
| `/var/cache/` | Cached application data that can be regenerated (APT package cache) |
| `/var/tmp/` | Temporary files that persist across reboots (unlike `/tmp`) |
| `/var/www/` | Web server document root (Apache/Nginx default) |
| `/var/mail/` | User mailboxes for local system mail |
| `/var/lock/` | Lock files indicating "this resource is in use" |

**Logs deep dive (`/var/log/`):**

Logs are indispensable for debugging. Key log files:

| Log File | Contains |
|---|---|
| `/var/log/syslog` (Debian) or `/var/log/messages` (RHEL) | General system messages |
| `/var/log/auth.log` (Debian) or `/var/log/secure` (RHEL) | Authentication attempts, sudo usage, SSH logins |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dpkg.log` | Package installation/removal history |
| `/var/log/apt/` | APT operation history |
| `/var/log/nginx/` | Nginx access and error logs |
| `/var/log/journal/` | Systemd's binary journal (use `journalctl` to read) |

```bash
# Read the system log in real time (follow mode, like tail -f)
sudo journalctl -f

# Show logs from a specific service
sudo journalctl -u nginx.service

# Show logs since last boot
sudo journalctl -b

# Show logs from the previous boot (useful after crashes)
sudo journalctl -b -1

# Show only error and above severity
sudo journalctl -p err

# Find failed SSH login attempts
grep "Failed password" /var/log/auth.log

# Monitor a log file in real time
sudo tail -f /var/log/nginx/error.log

# Check disk space used by logs
du -sh /var/log/*/ 2>/dev/null | sort -rh | head -10

# Clear the APT package cache (safe to do, frees significant space)
sudo apt clean

# View APT package installation history
cat /var/log/apt/history.log

# Check how much space Docker is using in /var/lib
du -sh /var/lib/docker/

# Rotate logs manually (useful if a log has gotten very large)
sudo logrotate -f /etc/logrotate.conf
```

---

### `/run` — Runtime Data (tmpfs)

`/run` is a **tmpfs** (temporary filesystem) — a filesystem that lives entirely in RAM rather than on disk. This means its contents are created fresh every time the system boots and are completely lost when the machine shuts down or restarts.

> **What is tmpfs?** tmpfs is a special filesystem type that uses RAM (and optionally swap space) as its storage medium. Files written to a tmpfs directory are stored in memory, which makes reads and writes extremely fast, but provides no persistence. It's used for data that only makes sense during the current session.

`/run` holds runtime state needed by the operating system and services from the moment they start up:

- **PID files**: Files named `<service>.pid` containing the Process ID of a running daemon (e.g., `/run/sshd.pid`). Other processes use these to know whether a service is running and how to communicate with it.
- **Unix sockets**: File-like objects used for inter-process communication. For example, `/run/docker.sock` is how the Docker CLI communicates with the Docker daemon.
- **Lock files**: Prevent multiple instances of a service from starting simultaneously.
- **Mount information**: Data about currently mounted filesystems.

```bash
# See what's currently in /run
ls /run

# Check what PID the SSH daemon is running as
cat /run/sshd.pid

# Check if Docker socket exists (indicates Docker daemon is running)
ls -la /run/docker.sock

# Verify /run is a tmpfs
df -T /run

# See all tmpfs mounts
df -T | grep tmpfs
```

---

### `/tmp` — Temporary Files

`/tmp` is for temporary files created by user applications and system processes during a session. On many modern Linux systems, `/tmp` is also a tmpfs (in-memory), but traditionally it lives on disk.

The key guarantee `/tmp` provides to applications is that they don't need to manage cleanup — files here are deleted automatically. On most systems this happens on reboot; with `systemd-tmpfiles`, old `/tmp` files can be cleaned by age.

> **`/tmp` vs `/var/tmp`:** `/tmp` may be cleared on every reboot. `/var/tmp` is for temporary data that should survive reboots — for example, the autosave state of a text editor, or a large file download that spans multiple sessions.

```bash
# Most applications use /tmp automatically, but you can use it too
TEMP_FILE=$(mktemp)            # creates a safe, unique temp file
TEMP_DIR=$(mktemp -d)          # creates a safe, unique temp directory
echo "working in: $TEMP_DIR"

# See what's in /tmp right now
ls -la /tmp

# Check the size of /tmp
df -h /tmp

# Find files in /tmp older than 1 day
find /tmp -mtime +1 -type f

# Check whether /tmp is a tmpfs (in-memory)
df -T /tmp

# Manually clean files in /tmp older than 2 days (safe on a running system)
sudo find /tmp -type f -atime +2 -delete
```

---

## 10. Storage & Mounts

### `/media` — Removable Media (Auto-mounted)

`/media` is the conventional mount point for removable media that the operating system manages automatically. When you plug in a USB drive, insert an SD card, or connect an external disk, the desktop environment (or `udisks` daemon) creates a subdirectory under `/media/<username>/` and mounts the device there.

```bash
# See what's currently mounted under /media
ls /media/$USER/

# Safely eject a USB drive (flush writes, then unmount)
udisksctl unmount -b /dev/sdb1
udisksctl power-off -b /dev/sdb

# Or the traditional way:
sudo umount /media/$USER/MY_USB

# Mount manually if auto-mount didn't work
sudo mount /dev/sdb1 /media/$USER/my_drive
```

---

### `/mnt` — Manual Mount Point

`/mnt` is a generic, manually managed mount point. You use it when you want to attach something yourself: an additional hard drive, a network share, a disk image, or a partition you're working on. Unlike `/media`, nothing auto-mounts here.

```bash
# Create a subdirectory in /mnt for clarity (good practice)
sudo mkdir -p /mnt/data_drive

# Mount a partition
sudo mount /dev/sdb1 /mnt/data_drive

# Mount a network share (NFS)
sudo mount -t nfs 192.168.1.100:/shared /mnt/nfs_share

# Mount a network share (SMB/Samba, Windows share)
sudo mount -t cifs //192.168.1.100/share /mnt/windows_share -o username=user

# Mount a disk image (.iso)
sudo mount -o loop,ro ~/ubuntu.iso /mnt/iso

# Check what's currently mounted
mount | grep /mnt
# or
df -h | grep /mnt

# Unmount when done
sudo umount /mnt/data_drive

# For a persistent mount that survives reboot, add to /etc/fstab (see /etc section)
```

---

## 11. Server Data

### `/srv` — Service Data

`/srv` is intended for data served *to the outside world* by services running on the machine. The FHS is explicit: this directory holds site-specific data used by services provided by the system, such as web server content or FTP files. The idea is to give operators a predictable, separate location for publicly served data, distinct from system files.

In practice, many distributions and administrators default to other locations (e.g., Nginx defaults to `/var/www/html/`), but `/srv` is the FHS-correct choice and is increasingly adopted.

```bash
# Set up a web root in /srv (FHS-correct pattern)
sudo mkdir -p /srv/www/mysite.com/public_html
sudo chown -R www-data:www-data /srv/www/

# Configure Nginx to serve from /srv
# In /etc/nginx/sites-available/mysite:
#   root /srv/www/mysite.com/public_html;

# Set up an FTP root
sudo mkdir -p /srv/ftp/public
sudo chmod 755 /srv/ftp/public

# Set up a Git repository server
sudo mkdir -p /srv/git/myrepo.git
sudo git init --bare /srv/git/myrepo.git
```

---

## 12. Distro-Specific Directories

### `/snap` — Snap Packages (Ubuntu)

`/snap` is specific to Ubuntu and other distributions that use Canonical's **Snap** package management system. Snap packages are self-contained application bundles that include all their dependencies — unlike traditional `.deb` or `.rpm` packages, which rely on system libraries. Each snap application runs in a confined sandbox with limited access to the host system.

Snap packages are mounted at `/snap/<appname>/<version>/` using a loop device. When you install a snap, a read-only filesystem image (`.snap` file, stored in `/var/lib/snapd/snaps/`) is mounted here.

> **The tradeoff with Snap:** The benefit of self-containment is that snaps work across different Ubuntu versions and never have dependency conflicts. The cost is disk space (each snap bundles its own libraries) and sometimes slower startup times compared to native packages.

```bash
# List all installed snap packages
snap list

# Install a snap package
sudo snap install code --classic   # VS Code (--classic allows more system access)

# Update all snaps
sudo snap refresh

# Remove a snap
sudo snap remove code

# See snap mount points
df -h | grep snap
ls /snap

# Check snap package details
snap info firefox
```

---

## 13. Quick Reference Summary

The table below distills the entire FHS into a single, scannable reference. The "FHS quadrant" column uses the shareable/non-shareable × static/variable framework from Section 1.

| Directory | Full Name | Primary Purpose | FHS Quadrant | Persistent? |
|---|---|---|---|---|
| `/` | Root | Top of the filesystem tree | — | Yes |
| `/home` | Home | Personal user files and config | Shareable / Variable | Yes |
| `/root` | Root Home | Superuser's home directory | Non-shareable / Variable | Yes |
| `/bin` | Binaries | Essential user commands | Shareable / Static | Yes |
| `/sbin` | System Binaries | Essential admin commands | Non-shareable / Static | Yes |
| `/usr` | Unix System Resources | User-installed apps and data | Shareable / Static | Yes |
| `/opt` | Optional | Self-contained third-party software | Shareable / Static | Yes |
| `/lib`, `/lib64` | Libraries | Shared libraries for /bin and /sbin | Shareable / Static | Yes |
| `/boot` | Boot | Bootloader and kernel images | Non-shareable / Static | Yes |
| `/sys` | Sysfs | Live kernel and hardware interface | Non-shareable / Variable | **No (RAM)** |
| `/proc` | Procfs | Process and kernel info | Non-shareable / Variable | **No (RAM)** |
| `/dev` | Devices | Device files for hardware | Non-shareable / Variable | **No (RAM)** |
| `/etc` | Et cetera | System-wide configuration | Non-shareable / Static | Yes |
| `/var` | Variable | Logs, databases, caches, spools | Shareable+Non / Variable | Yes |
| `/run` | Runtime | PID files, sockets (since last boot) | Non-shareable / Variable | **No (RAM)** |
| `/tmp` | Temporary | Session-scoped temp files | Non-shareable / Variable | Usually No |
| `/media` | Media | Auto-mounted removable drives | — | While mounted |
| `/mnt` | Mount | Manually mounted filesystems | — | While mounted |
| `/srv` | Service | Data served by this machine | Shareable / Variable | Yes |
| `/snap` | Snap | Ubuntu snap package mounts | — | While installed |

---

*This document follows the Linux Foundation's File Hierarchy Standard (FHS) 3.0. For the authoritative specification, see: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html*


**Understand the why** — Each directory embodies a semantic commitment:
   - `/bin` = "I boot the OS"
   - `/etc` = "I'm a system policy"
   - `/var/lib` = "I'm data you can't regenerate"
   - `/var/cache` = "I'm a speedup you can throw away"
   - `/var/log` = "I'm an audit record"
   - `/run` = "I live until the next reboot"
   - `/tmp` = "I'm garbage you can delete anytime"


# Linux Filesystem: Complete Production Taxonomy
## Including ML, Data Apps, Caches, Web Services, and Distributed Systems

---

## DECISION AXES (MECE)

### Primary Axis: Static vs Dynamic
- **Static**: Compiled, built, fixed at deployment time
- **Dynamic**: Generated at runtime, mutable, grows over time

### Secondary Axis: Ownership
- **System** (root, distro-managed)
- **Service/Daemon** (app-specific user like `postgres`, `www-data`)
- **User** (individual user home)

### Tertiary Axis: Lifecycle
- **Persistent**: Survives reboots indefinitely
- **Boot-cycle**: Exists until next reboot, cleaned on shutdown
- **Ephemeral**: Safe to delete anytime

### Quaternary Axis: Content Type
- Executables
- Libraries & dependencies
- Configuration
- Data (persistent state)
- Cache (derived, regenerable)
- Logs (audit trail)
- Runtime metadata (PIDs, sockets)
- Read-only reference data (code examples, docs, fonts, assets)
- User-served content (web pages, uploads)
- Models & large inference files

---

## COMPLETE DIRECTORY TAXONOMY

### BINARIES & EXECUTABLES

#### `/bin` and `/sbin`
**Semantics**: System boot-critical executables
- **Contents**: ls, cat, mount, init, systemctl, basic shell utilities
- **Owner**: OS distribution (root)
- **Lifecycle**: Persistent; replaced on OS upgrade
- **Decision trigger**: "Do I need this tool before /usr mounts?"
- **Example**: `ls` is in /bin because it's needed in single-user mode
- **Production note**: Most modern systems symlink to /usr/bin

#### `/usr/bin` and `/usr/sbin`
**Semantics**: Distribution-provided application binaries
- **Contents**: git, python, nginx, postgresql, docker, npm
- **Owner**: Package manager (apt, yum, pacman)
- **Lifecycle**: Persistent; replaced when package is upgraded
- **Decision trigger**: "Is this provided by a distro package?"
- **Example**: `apt install nginx` → `/usr/bin/nginx`
- **Production note**: Conflict zone: manual upgrades vs package manager

#### `/usr/local/bin` and `/usr/local/sbin`
**Semantics**: Locally-compiled or manually-installed executables (survives distro upgrades)
- **Contents**: go, rustc, internal-deploy (compiled from source), tools built by your team
- **Owner**: System administrator or CI/CD pipeline
- **Lifecycle**: Persistent; **preserved across distro upgrades**
- **Decision trigger**: "Did I compile this locally, and should it survive distro upgrades?"
- **Example**: 
  ```bash
  wget https://go.dev/dl/go1.21.linux-amd64.tar.gz
  tar -C /usr/local -xzf go1.21.linux-amd64.tar.gz
  # Go binary now at /usr/local/go/bin/go
  ln -s /usr/local/go/bin/go /usr/local/bin/go
  ```
- **Backup**: Track in version control (not in binary form, but build scripts)

#### `/opt/{vendor}/{app}`
**Semantics**: Large, self-contained vendor applications with isolation
- **Contents**: elasticsearch, mongodb, datadog-agent, custom large apps (100+ files with internal dependencies)
- **Owner**: Vendor installer or custom deployment script
- **Lifecycle**: Persistent; **complete isolation** from OS package ecosystem
- **Decision trigger**: "Is this a large, self-contained app with >50 files and internal versioning?"
- **Structure**:
  ```
  /opt/elasticsearch/
    ├── bin/elasticsearch
    ├── bin/elasticsearch-plugin
    ├── config/elasticsearch.yml
    ├── data/nodes/
    ├── logs/elasticsearch.log
    ├── plugins/
    ├── lib/  (internal dependencies)
    └── LICENSE.txt
  ```
- **Why isolated**: Uninstall = `rm -rf /opt/elasticsearch/` (clean, no leftover files)
- **Production pattern**: Symlink for convenience:
  ```bash
  ln -s /opt/elasticsearch/bin/elasticsearch /usr/local/bin/elasticsearch
  ```
- **Example apps**: 
  - Large databases: MongoDB, Cassandra, DataStax Astra
  - Search engines: Elasticsearch, Solr, OpenSearch
  - Distributed message brokers: Kafka, RabbitMQ
  - ML platforms: MLflow, Kubeflow
  - Monitoring: DataDog agent, New Relic agent
  - Custom proprietary tools

---

### LIBRARIES & DEPENDENCIES

#### `/lib`, `/lib64`
**Semantics**: System libraries needed for core OS functionality
- **Contents**: libc.so, libssl.so (SSL/TLS), libcrypto.so
- **Owner**: OS distribution
- **Lifecycle**: Persistent; replaced on OS upgrade
- **Decision trigger**: "Is this a core system library needed at boot?"

#### `/usr/lib`, `/usr/lib64`
**Semantics**: Distribution-provided application libraries
- **Contents**: Python packages via apt, shared object files (.so), plugins
- **Owner**: Package manager
- **Lifecycle**: Persistent; replaced when package upgraded
- **Example**: 
  ```
  /usr/lib/python3/dist-packages/  (system Python packages from apt)
  /usr/lib/x86_64-linux-gnu/       (compiled libraries)
  ```

#### `/usr/local/lib`, `/usr/local/lib64`
**Semantics**: Locally-compiled libraries (survives distro upgrades)
- **Contents**: Custom-built libraries, libraries compiled from source
- **Owner**: System administrator
- **Lifecycle**: Persistent; preserved across upgrades
- **Example**:
  ```bash
  ./configure --prefix=/usr/local
  make
  make install
  # Libraries go to /usr/local/lib
  ```

---

### CONFIGURATION (Policy)

#### `/etc/{app}`
**Semantics**: System-wide configuration files (affects all processes and users)
- **Contents**: nginx.conf, postgresql.conf, mysql.cnf, redis.conf
- **Owner**: Root (administrator-edited)
- **Lifecycle**: Persistent across reboots; preserved on package upgrade (with dpkg prompts)
- **Permissions**: Usually 644 (readable by all, writable by root only)
- **Decision trigger**: "Is this a global policy that affects the whole system?"
- **Example structure**:
  ```
  /etc/nginx/
    ├── nginx.conf           (main config)
    ├── conf.d/              (additional site configs)
    ├── sites-available/
    └── sites-enabled/       (symlinks to active sites)
  
  /etc/postgresql/
    ├── postgresql.conf      (main)
    ├── pg_hba.conf          (authentication)
    └── environment
  
  /etc/redis/
    └── redis.conf
  ```
- **Backup strategy**: YES, version control all /etc/{app} (these are policies)
- **Cluster deployment**: /etc configs are usually templated by Ansible/Terraform and deployed identically across all nodes

#### `~/.config/{app}`
**Semantics**: Per-user, application-specific preferences (doesn't affect other users)
- **Contents**: VSCode settings, Git config, shell config
- **Owner**: Individual user
- **Lifecycle**: Persistent; survives OS upgrades
- **Decision trigger**: "Does each user customize this independently, or is it system-wide?"
- **Example**:
  ```
  ~/.config/git/config          (per-user Git settings)
  ~/.config/vscode/settings.json (per-user VSCode)
  ~/.bashrc, ~/.zshrc           (per-user shell config)
  ```
- **Note**: Most system **daemons** (nginx, postgres, redis) don't source ~/.config because they run as service users (www-data, postgres) that don't have home directories

#### `/run/{app}.conf` or `/etc/systemd/system/{app}.service.d/`
**Semantics**: Runtime overrides or systemd service configuration
- **Contents**: systemd service files, systemd drop-in directives
- **Use case**: Override ExecStart, set environment variables, modify resource limits
- **Example**:
  ```ini
  [Service]
  Environment="DATABASE_URL=postgres://..."
  MemoryLimit=2G
  CPUQuota=50%
  ```

---

### DATA & STATE (Persistent)

#### `/var/lib/{app}`
**Semantics**: Persistent state the application **requires** to function (source of truth)
- **Contents**: Database files, app state, internal data structures
- **Owner**: Service user (postgres, mysql, mongodb, redis)
- **Lifecycle**: Persistent across reboots; **never auto-deleted**
- **Backup**: YES, **full backup required** (this is the source of truth)
- **Decision trigger**: "Would deleting this cause data loss?"
- **Examples**:
  ```
  /var/lib/postgresql/data/      (database tables, indexes, WAL)
  /var/lib/mysql/                (database files)
  /var/lib/mongodb/              (MongoDB data)
  /var/lib/redis/                (Redis snapshots)
  /var/lib/apt/cache/            (apt package cache; note: this is cached, not source of truth)
  /var/lib/docker/               (Docker container storage, images)
  /var/lib/elasticsearch/        (Elasticsearch indices and shards)
  ```
- **Permissions**: Usually 700 or 750 (only service user can read/write)
- **Storage planning**: Often mounted on separate, high-reliability storage
  ```bash
  # Mount on fast NVMe or reliable SAN for database performance
  mount /dev/nvme0n1 /var/lib/postgresql
  mount /dev/sda1 /var/lib/elasticsearch  # Separate spindle for I/O parallelism
  ```

#### `/var/cache/{app}`
**Semantics**: Derived data that **can be regenerated** (not source of truth)
- **Contents**: Computed recommendations, thumbnails, compiled assets, parsed config
- **Owner**: Service user (same as app)
- **Lifecycle**: Persistent; **safe to delete**, app rebuilds on demand
- **Backup**: NO (skip; cache is reconstructible)
- **Decision trigger**: "Would deleting this slow down the app, but not break it?"
- **Cleanup**: Manual or cron-based (e.g., `find /var/cache/myapp -mtime +30 -delete`)
- **Examples**:
  ```
  /var/cache/apt/archives/       (downloaded .deb packages; can be purged)
  /var/cache/myapp/thumbnails/   (image thumbnails; regenerated on access)
  /var/cache/recommendation-engine/models/  (computed embeddings; can be re-run)
  /var/cache/web-server/compiled/  (precompiled CSS/JS; can be rebuilt)
  /var/cache/build/              (intermediate build artifacts)
  ```
- **Storage planning**: Can be on slower storage, can be cleaned aggressively
  ```bash
  # Safe to use local SSDs that are dropped during instance shutdown
  mount /dev/nvme1n1 /var/cache/myapp
  ```

---

### LOGS (Audit Trail)

#### `/var/log/{app}`
**Semantics**: Application output and audit trail (transient records of what happened)
- **Contents**: access logs, error logs, debug logs, application stdout/stderr
- **Owner**: Service user or root (written to by app, rotated by logrotate)
- **Lifecycle**: Persistent but **auto-cleaned** (compressed, deleted after N days)
- **Backup**: OPTIONAL (archive for compliance/audit, but not required for app recovery)
- **Decision trigger**: "Is this a record of what happened, not state the app needs?"
- **Cleanup**: Via `logrotate`, compressed daily/weekly, deleted after 7–30 days
- **Examples**:
  ```
  /var/log/nginx/
    ├── access.log           (HTTP requests)
    ├── error.log            (HTTP errors)
    ├── access.log.1.gz      (rotated, compressed)
    └── access.log.7.gz      (oldest, next to be deleted)
  
  /var/log/postgresql/
    ├── postgresql.log       (query logs, connection info)
    └── postgresql-2024-03-01.log.gz
  
  /var/log/redis/
    └── redis.log            (server events, slowlog)
  
  /var/log/mongodb/
    └── mongod.log           (operation logs)
  ```
- **Volume**: Can be enormous (Nginx: 100+ MB/day on busy sites)
- **Storage**: Separate filesystem to avoid filling `/var` and breaking database
  ```bash
  mount /dev/sdb1 /var/log  # Slower storage OK; logs are sequential I/O
  ```
- **Configuration**:
  ```bash
  # /etc/logrotate.d/nginx
  /var/log/nginx/*.log {
    daily
    missingok
    rotate 14        # keep 14 days
    compress
    notifempty
    create 0640 www-data adm
  }
  ```

#### `/run/log/journal` (Systemd journal)
**Semantics**: Systemd-managed logs for services launched by systemd
- **Contents**: Service startup/shutdown, application logs if journald-enabled
- **Lifecycle**: Volatile (tmpfs-backed); cleared on reboot
- **Query**: `journalctl -u nginx` (view logs for nginx service)
- **Persistence**: Optionally persist to `/var/log/journal` if configured

---

### RUNTIME STATE (Boot-transient, ephemeral)

#### `/run/{app}.pid`
**Semantics**: Process ID file (used by init scripts to manage the app)
- **Contents**: PID of the main process
- **Owner**: Service user
- **Lifecycle**: Created at startup, deleted at shutdown; cleared on reboot
- **Storage**: tmpfs (RAM-backed)
- **Purpose**: 
  - Init scripts use it to stop/restart the app
  - Prevents duplicate instances from starting
  - Monitoring tools query it to check if app is running
- **Examples**:
  ```
  /run/nginx.pid
  /run/postgresql.pid
  /run/redis.pid
  /run/mongodb.pid
  ```

#### `/run/{app}/socket` (Unix domain sockets)
**Semantics**: IPC endpoint for inter-process communication
- **Contents**: Socket file (special file type)
- **Owner**: Service user
- **Lifecycle**: Created at startup, deleted on shutdown; cleared on reboot
- **Storage**: tmpfs
- **Purpose**: 
  - Apps communicate without network overhead
  - More secure than TCP (filesystem permissions)
  - Lower latency than network sockets
- **Examples**:
  ```
  /run/dbus/system_bus_socket        (D-Bus system socket)
  /run/postgresql/.s.PGSQL.5432      (PostgreSQL Unix socket)
  /run/redis.sock                    (Redis Unix socket)
  /run/docker.sock                   (Docker daemon socket)
  ```
- **Permissions**: Usually 660 or 666 (group-writable for clients)

#### `/run/lock/` (Legacy lockfiles)
**Semantics**: Mutual exclusion for accessing shared resources
- **Contents**: Lock files (often empty, presence indicates "locked")
- **Owner**: Service user
- **Lifecycle**: Cleared on reboot
- **Note**: Superseded by `/run/{app}.pid` and systemd
- **Deprecated**: Most modern apps use `/run/{app}.pid`

---

### EPHEMERAL STORAGE

#### `/tmp`
**Semantics**: Scratch space; any process can write; safe to delete anytime
- **Contents**: Temporary files, upload buffers, build artifacts, temporary caches
- **Owner**: Any user/process
- **Lifecycle**: Cleared daily by `tmpwatch` or `systemd-tmpfiles`
- **Permissions**: 1777 (sticky bit: users can delete only their own files)
- **Storage**: Often tmpfs (RAM) but can be on disk
- **Decision trigger**: "Is this ephemeral and safe to lose at any time?"
- **Cleanup**: Automatic; files older than 10–30 days deleted
- **When to use**:
  ```
  # Build artifacts
  /tmp/build_xyz/          (compilation temporary files)
  
  # Upload buffers
  /tmp/upload_buffer_12345  (file chunk during upload)
  
  # Session data
  /tmp/session_data/       (temporary session storage)
  
  # Process-specific temps
  /tmp/python_abc123/      (Python tempfile)
  ```
- **Pitfall**: If app writes socket to /tmp and tmpwatch deletes it after 7 days, app still running but socket gone → clients can't connect
  - **Solution**: Write sockets to `/run` instead

#### `/var/tmp`
**Semantics**: Slower cleanup than /tmp; for larger or longer-lived temporary files
- **Contents**: VM image building, large scratch files, data that should survive a few days
- **Owner**: Any user/process
- **Lifecycle**: Cleared after 30 days (longer than /tmp)
- **Storage**: Always disk (not tmpfs)
- **Decision trigger**: "Is this temporary but needs to survive longer than /tmp?"
- **Examples**:
  ```
  /var/tmp/packer_build/     (Packer VM image building)
  /var/tmp/large_download/   (partial downloads)
  ```

#### `/dev/shm`
**Semantics**: POSIX shared memory; tmpfs-backed RAM disk
- **Contents**: Named shared memory objects, memory-mapped files
- **Owner**: Any user (usually same user for IPC)
- **Lifecycle**: Cleared on reboot
- **Storage**: RAM
- **Use case**: Fast IPC between processes (faster than Unix sockets, no disk I/O)
- **Examples**:
  ```
  # Python multiprocessing shared arrays
  /dev/shm/mp_queue_12345
  
  # Databases doing shared buffer pools
  /dev/shm/postgresql_shared_memory
  ```
- **Caution**: Takes RAM; large /dev/shm usage can OOM the system

---

### VIRTUAL FILESYSTEMS (Read-only views of kernel state)

#### `/proc`
**Semantics**: Process information as files (VFS, not on-disk)
- **Contents**: Per-process directories (/proc/[pid]/), kernel info (/proc/meminfo, /proc/cpuinfo)
- **Owner**: Kernel (read-only)
- **Lifecycle**: Virtual; exists while kernel is running
- **Query**: `cat /proc/meminfo`, `cat /proc/cpuinfo`, `cat /proc/[pid]/status`
- **Examples**:
  ```
  /proc/1234/cmdline         (command-line of process 1234)
  /proc/1234/environ         (environment variables)
  /proc/1234/fd/             (open file descriptors)
  /proc/1234/maps            (memory mappings)
  /proc/meminfo              (RAM usage)
  /proc/cpuinfo              (CPU details)
  /proc/diskstats            (disk I/O stats)
  /proc/net/tcp              (TCP connections)
  ```
- **Use**: Monitoring, debugging, process introspection

#### `/sys`
**Semantics**: Kernel subsystems and device information (VFS)
- **Contents**: Device info, kernel parameters, power management
- **Owner**: Kernel
- **Lifecycle**: Virtual
- **Write**: Writable for tuning kernel parameters (requires root)
- **Examples**:
  ```
  /sys/devices/              (device tree)
  /sys/kernel/               (kernel parameters)
  /sys/class/net/            (network interfaces)
  /sys/block/                (block devices)
  ```
- **Tuning**:
  ```bash
  # Increase TCP backlog
  echo 2048 > /proc/sys/net/ipv4/tcp_max_syn_backlog
  # Or: sysctl -w net.ipv4.tcp_max_syn_backlog=2048
  ```

---

### READ-ONLY REFERENCE DATA

#### `/usr/share/doc`
**Semantics**: Package documentation (man pages, README files)
- **Contents**: Upstream documentation, license files, examples
- **Owner**: OS distribution
- **Lifecycle**: Persistent; replaced on package upgrade
- **Size**: Large (can be 500 MB+ on a full installation)
- **Example**: `cat /usr/share/doc/nginx/README.Debian`

#### `/usr/share/man`
**Semantics**: Man pages for command documentation
- **Contents**: Manual pages for every system command and library
- **Owner**: OS distribution
- **Query**: `man nginx`, `man postgresql`, `man 2 open`
- **Purpose**: Built-in documentation (no internet needed)

#### `/usr/share/info`
**Semantics**: GNU info documentation
- **Query**: `info coreutils`

#### `/usr/share/fonts`
**Semantics**: System fonts for rendering text
- **Contents**: TrueType, OpenType, bitmap fonts
- **Owner**: OS distribution (user fonts in ~/.fonts)
- **Use**: Desktop apps, web servers serving custom fonts
- **Examples**:
  ```
  /usr/share/fonts/truetype/
  /usr/share/fonts/opentype/
  ```

#### `/usr/share/pixmaps`, `/usr/share/icons`
**Semantics**: Icons and images for UI
- **Contents**: Application icons, desktop icons, emojis
- **Owner**: OS distribution + package icons
- **Use**: Desktop environments, web UIs that ship icons

#### `/usr/share/applications`
**Semantics**: Desktop application metadata (.desktop files)
- **Contents**: Application launchers for desktop environments
- **Example**:
  ```ini
  # /usr/share/applications/firefox.desktop
  [Desktop Entry]
  Name=Firefox
  Exec=firefox %u
  Icon=firefox
  ```

#### `/usr/share/ca-certificates`
**Semantics**: CA certificates for SSL/TLS verification
- **Contents**: Root certificates (*.crt files)
- **Owner**: OS distribution
- **Use**: HTTPS validation, package manager certificate verification
- **Update**: `update-ca-certificates` (regenerates /etc/ssl/certs)

#### `/usr/share/zoneinfo`
**Semantics**: Timezone database
- **Contents**: Timezone definitions (tzdata)
- **Owner**: OS distribution
- **Use**: Time/date libraries to convert between zones
- **Query**: `cat /usr/share/zoneinfo/America/New_York`

---

### USER-SERVED DATA (Web content)

#### `/var/www/`
**Semantics**: Default location for web server content
- **Contents**: HTML, CSS, JS, static images served by Nginx/Apache
- **Owner**: www-data user (web server) or specific app user
- **Lifecycle**: Persistent
- **Permissions**: 755 (readable by all, writable by owner)
- **Structure**:
  ```
  /var/www/html/                 (default root)
  /var/www/myapp/                (per-app structure)
    ├── public/                  (static files)
    ├── uploads/                 (user uploads)
    └── sessions/                (session data)
  ```
- **Backup**: YES (user content)
- **Decision trigger**: "Is this content served to end-users?"

#### `/srv/`
**Semantics**: Service data directory (alternative to /var/www)
- **Contents**: Web content, FTP files, Git repos, user-served data
- **Owner**: Service user or root
- **Lifecycle**: Persistent
- **Decision trigger**: "Is this data for a service but not a standard web app?"
- **Structure**:
  ```
  /srv/www/                      (web content)
  /srv/ftp/                      (FTP server files)
  /srv/git/                      (Git repositories)
  /srv/data/myapp/               (per-app data)
  ```
- **When to prefer /srv over /var/www**: 
  - Multiple services serving data
  - Custom applications (not standard web apps)
  - Emphasis on "data for this service" rather than "web root"
- **Example**:
  ```
  /srv/nextcloud/data/           (user files)
  /srv/diaspora/uploads/         (social network uploads)
  /srv/git-server/repos/         (Git repositories)
  ```
- **Backup**: YES (user content)

---

### ML & DATA SCIENCE SPECIFIC

#### `/var/lib/ml/models/`
**Semantics**: Trained ML models (large binary files, source of truth for inference)
- **Contents**: PyTorch .pth files, TensorFlow SavedModel, ONNX models, weights
- **Owner**: ML service user or root
- **Lifecycle**: Persistent
- **Backup**: YES (critical; models take weeks to train)
- **Storage**: Large (can be 500 MB – 100 GB+ for LLMs)
- **Structure**:
  ```
  /var/lib/ml/models/
    ├── bert-base-uncased/        (model directory)
    ├── gpt2/
    └── custom-classifier-v3/
  ```
- **Decision trigger**: "Is this a trained model needed for inference?"
- **Best practice**: Version control model paths
  ```bash
  /var/lib/ml/models/bert-base-uncased/pytorch_model.bin
  # Keep version in path or symlink to latest:
  ln -s bert-base-uncased.v2.0 bert-base-uncased-latest
  ```

#### `/var/cache/ml/datasets/`
**Semantics**: Training datasets (large, preprocessed, can be regenerated)
- **Contents**: Training data files, preprocessed numpy arrays, parquet files
- **Owner**: ML training service
- **Lifecycle**: Persistent; safe to delete (can re-download or preprocess)
- **Backup**: NO (regenerable; keep original raw data elsewhere)
- **Storage**: Very large (can be 100s of GB)
- **Decision trigger**: "Is this preprocessed data for training (not the trained model)?"
- **Structure**:
  ```
  /var/cache/ml/datasets/
    ├── imagenet/               (can be re-downloaded)
    ├── wikitext/               (preprocessed corpus)
    └── my-custom-dataset.pkl
  ```
- **Cleanup**: Safe to `rm -rf /var/cache/ml/datasets/` and re-download

#### `/var/lib/ml/experiments/`
**Semantics**: Training run metadata and checkpoints
- **Contents**: Experiment logs, tensorboard events, model checkpoints during training
- **Owner**: ML training service
- **Lifecycle**: Persistent (until training complete, then archive)
- **Backup**: MAYBE (depends on if you want to resume interrupted training)
- **Structure**:
  ```
  /var/lib/ml/experiments/
    ├── exp-001-lr-0.001/
    │   ├── checkpoints/
    │   │   ├── epoch-10.pth
    │   │   └── epoch-20.pth (best)
    │   ├── logs/
    │   │   └── events.out.tfevents...
    │   └── config.json
    └── exp-002-lr-0.0001/
  ```

#### `/var/cache/ml/feature-store/`
**Semantics**: Precomputed features for ML (cached derived features)
- **Contents**: Feature vectors, embeddings, aggregated statistics
- **Owner**: ML service
- **Lifecycle**: Persistent; safe to delete (recomputed on demand)
- **Backup**: NO (cache)
- **Decision trigger**: "Are these computed features used by models (not trained models)?"

---

### DISTRIBUTED SYSTEMS & DATABASES

#### `/var/lib/postgresql/`
**Semantics**: PostgreSQL database files
- **Structure**:
  ```
  /var/lib/postgresql/
    ├── data/                      (actual data)
    │   ├── base/                  (tables)
    │   ├── pg_wal/                (write-ahead logs for recovery)
    │   └── pg_xact/               (transaction status)
    ├── pg_stat_tmp/               (statistics)
    └── backups/                   (backup directory)
  ```
- **Backup**: YES, full backup + WAL archival
- **Storage**: Separate fast disk (NVMe preferred)
- **Permissions**: 700 (postgres user only)

#### `/var/lib/mysql/` or `/var/lib/mariadb/`
**Semantics**: MySQL/MariaDB database files
- **Structure**:
  ```
  /var/lib/mysql/
    ├── mysql/                     (system tables)
    ├── your_database/
    │   ├── table1.ibd             (InnoDB table)
    │   ├── table2.ibd
    │   └── table2.MYI             (MyISAM metadata)
    ├── ib_logfile0                (redo log)
    └── ibdata1                    (shared tablespace)
  ```
- **Backup**: YES, mysqldump or XtraBackup
- **Storage**: Separate disk

#### `/var/lib/mongodb/`
**Semantics**: MongoDB data directory
- **Structure**:
  ```
  /var/lib/mongodb/
    ├── collection-0-5439...       (data files)
    ├── collection-1-5439...
    ├── index-3-5439...
    ├── diagnostic.data/           (diagnostics)
    └── journal/                   (write-ahead journal)
  ```
- **Backup**: YES, mongodump or snapshots
- **Storage**: Separate disk

#### `/var/lib/redis/`
**Semantics**: Redis dump and AOF (append-only file)
- **Contents**:
  ```
  /var/lib/redis/
    ├── dump.rdb                   (RDB snapshot)
    ├── appendonly.aof             (AOF journal)
    └── temp-rewrite-aof           (AOF rewrite temp)
  ```
- **Backup**: MAYBE (Redis is often cache; depends on persistence mode)

#### `/var/lib/elasticsearch/`
**Semantics**: Elasticsearch shards and indices
- **Structure**:
  ```
  /var/lib/elasticsearch/
    ├── nodes/
    │   ├── 0/
    │   │   ├── indices/           (all indices)
    │   │   │   └── my_index/
    │   │   │       └── 0/
    │   │   │           ├── index/
    │   │   │           └── translog/
    │   │   └── _state/            (node metadata)
    │   └── 1/
    └── repository-backup/         (snapshot repo)
  ```
- **Backup**: YES, via snapshots
- **Storage**: Very large, separate fast disk

#### `/var/lib/kafka/`
**Semantics**: Kafka log segments (partitions)
- **Structure**:
  ```
  /var/lib/kafka/
    ├── topic1-0/
    │   ├── 00000000000000000000.log
    │   ├── 00000000000000000000.index
    │   └── 00000000000000000000.timeindex
    ├── topic2-0/
    └── recovery-point-offset-checkpoint
  ```
- **Backup**: MAYBE (depends on replication factor and retention policy)
- **Storage**: Very large, separate fast disk, high IOPS

#### `/var/lib/docker/`
**Semantics**: Docker containers, images, volumes
- **Structure**:
  ```
  /var/lib/docker/
    ├── containers/               (running container data)
    ├── images/                   (image layers)
    ├── volumes/                  (persistent volumes)
    ├── overlay2/                 (union filesystem layers)
    └── buildkit/
  ```
- **Backup**: MAYBE (depends on what's in containers; typically ephemeral)
- **Storage**: Can be very large

---

### SPECIFIC APP PATTERNS

#### Python/Node.js Web App (`/var/lib/myapp/` structure)
```
/var/lib/myapp/
  ├── venv/                     (Python virtual env)
  ├── node_modules/             (Node deps)
  ├── data/
  │   ├── db.sqlite3            (or postgres at /var/lib/postgresql/)
  │   ├── uploads/              (user files)
  │   └── session_store/        (session data)
  └── cache/                    (moved to /var/cache/myapp/)

/var/cache/myapp/
  ├── compiled_assets/          (CSS/JS)
  ├── thumbnails/
  └── redis_cache/              (if using local Redis cache)

/var/log/myapp/
  ├── app.log
  ├── access.log
  └── error.log

/run/myapp/
  ├── myapp.pid
  ├── myapp.sock
  └── myapp.conf (runtime config)

/etc/myapp/
  ├── config.yaml
  ├── settings.json
  └── secrets.env (or vault)
```

#### ML Training Pipeline
```
/var/lib/ml/
  ├── models/                  (trained models)
  ├── experiments/             (training runs)
  │   ├── exp-001/
  │   │   ├── checkpoints/
  │   │   ├── logs/
  │   │   └── config.json
  │   └── exp-002/
  └── pipelines/               (data pipeline code)

/var/cache/ml/
  ├── datasets/                (preprocessed training data)
  ├── features/                (computed features)
  └── embeddings/              (precomputed embeddings)

/var/log/ml/
  ├── training.log             (training progress)
  ├── inference.log            (inference requests)
  └── data_pipeline.log

/run/ml/
  ├── training-job-1.pid
  └── inference-server.sock

/etc/ml/
  ├── hyperparameters.yaml
  ├── feature_config.json
  └── model_registry.yaml
```

---

## DECISION MATRIX: WHERE DOES IT GO?

| Question | Answer | Location |
|----------|--------|----------|
| Is it an executable? | Boot-critical | `/bin` |
| | Distro package | `/usr/bin` |
| | Locally compiled | `/usr/local/bin` |
| | Large vendor app | `/opt/{vendor}/{app}` |
| Is it a library? | System lib | `/lib` |
| | Distro lib | `/usr/lib` |
| | Local lib | `/usr/local/lib` |
| Is it configuration? | System-wide policy | `/etc/{app}` |
| | Per-user settings | `~/.config/{app}` |
| Is it persistent data? | Source of truth (can't regenerate) | `/var/lib/{app}` |
| | Derived, regenerable | `/var/cache/{app}` |
| | Trained ML model | `/var/lib/ml/models/` |
| | Training data | `/var/cache/ml/datasets/` |
| | Database files | `/var/lib/{db}/` |
| Is it a log? | App audit trail | `/var/log/{app}` |
| | Transient logs | `/run/log/` (journald) |
| Is it runtime state? | PID, socket, IPC | `/run/{app}/` |
| | Boot-transient | `/tmp/` or `/var/tmp/` |
| | Shared memory | `/dev/shm/` |
| Is it served to users? | Web content | `/var/www/` or `/srv/` |
| | FTP files | `/srv/ftp/` |
| | Git repos | `/srv/git/` |
| Is it reference data? | Documentation | `/usr/share/doc/` |
| | Man pages | `/usr/share/man/` |
| | Fonts | `/usr/share/fonts/` |
| | Icons | `/usr/share/icons/` |
| | CA certs | `/usr/share/ca-certificates/` |
| Is it kernel state? | Process info | `/proc/` (read-only) |
| | Kernel params | `/sys/` (readable, writable for tuning) |

---

## PRODUCTION LAYOUT EXAMPLE: ML-POWERED WEB APP WITH INFERENCE

```
Application: recommendation-engine (web service + ML inference)

Binaries:
  /usr/local/bin/recommendation-engine  ← wrapper script

Source code:
  /opt/recommendation-engine/
    ├── app/
    ├── models/
    ├── requirements.txt
    └── setup.sh

Configuration:
  /etc/recommendation-engine/
    ├── config.yaml              (system config)
    ├── hyperparameters.json
    ├── secrets.vault            (encrypted via Vault)
    └── systemd/recommendation-engine.service

Persistent data:
  /var/lib/recommendation-engine/
    ├── models/                  (trained models)
    │   ├── bert-model/
    │   └── classifier-v3/
    ├── user-embeddings/         (user computed embeddings)
    └── state.db                 (app state)

Cache:
  /var/cache/recommendation-engine/
    ├── feature-cache/           (computed features; regenerable)
    ├── query-cache/             (recent recommendations cache)
    └── compiled-assets/         (CSS/JS)

Logs:
  /var/log/recommendation-engine/
    ├── app.log                  (rotated daily)
    ├── inference.log            (model inference timing)
    ├── training.log             (model training runs)
    └── access.log               (HTTP requests)

Runtime:
  /run/recommendation-engine/
    ├── app.pid
    ├── inference.sock           (IPC for inference calls)
    └── state.lock

Database:
  /var/lib/postgresql/
    └── recommendation_db/       (user data, model metadata)

Monitoring:
  /var/lib/prometheus/           (if using Prometheus)
  /var/lib/grafana/              (if using Grafana)

Backup strategy:
  ✓ /var/lib/recommendation-engine/models/     (trained models)
  ✓ /var/lib/postgresql/                       (user data)
  ✓ /etc/recommendation-engine/                (configs)
  ✗ /var/cache/recommendation-engine/          (regenerable)
  ✗ /var/log/                                  (transient)
  ✗ /run/                                      (ephemeral)
```

---

## KEY SEMANTIC RULES (TL;DR)

1. **Executables encode trust hierarchy**: `/bin` (OS) → `/usr/bin` (distro) → `/usr/local/bin` (admin) → `/opt` (vendor)

2. **Data vs Logs**: 
   - `/var/lib` = stuff the app needs to function (BACK IT UP)
   - `/var/cache` = speedups that can be recalculated (DON'T BACK UP)
   - `/var/log` = audit trail of what happened (ARCHIVE, DON'T BACK UP as primary)

3. **Persistence vs Ephemeral**:
   - `/tmp` = deleted anytime; OK to lose
   - `/run` = cleared on reboot; needed during boot cycle
   - `/var/lib` = survives indefinitely; never auto-cleaned

4. **System vs User**:
   - `/etc` = system policy (affects everyone)
   - `~/.config` = user preference (only affects that user)

5. **Serving data**:
   - `/var/www` = standard web root (default)
   - `/srv` = service data (non-web or multiple services)

6. **ML-specific**:
   - Models → `/var/lib/ml/models/` (backup them!)
   - Training data → `/var/cache/ml/datasets/` (regenerable)
   - Experiments → `/var/lib/ml/experiments/` (maybe backup)
   - Feature cache → `/var/cache/ml/features/` (skip backup)

7. **Distributed systems**:
   - Database data → `/var/lib/{db}/` on separate fast disk
   - Logs → `/var/log/` on separate disk (can be slower)
   - Replication/WAL → `/var/lib/{db}/wal/` or `/var/lib/{db}/journal/`

---

**The core principle**: By choosing the right directory, you declare the file's **ownership**, **lifecycle**, and **backup strategy**. This makes systems predictable and maintainable.