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

To understand why Linux organizes its files the way it does, it helps to contrast it with Windows — the other dominant operating system most people are familiar with.

**The DOS/Windows lineage** begins with MS-DOS (Microsoft Disk Operating System), a command-line-only environment that predates graphical interfaces. In DOS, everything revolves around *drives* identified by letters. The letters `A:` and `B:` were reserved for floppy disk drives — physical removable disks that you could slot in and out of a machine. When hard drives arrived, `C:` became the conventional name for the primary internal drive, and additional drives or partitions followed alphabetically (`D:`, `E:`, and so on). This letter-based scheme stuck. Windows started as a graphical desktop environment (a GUI) that ran *on top of* MS-DOS — you literally typed `win` at the DOS prompt to launch it. Over time, from Windows NT onward, Microsoft gradually replaced DOS with a fully independent kernel, but the drive-letter convention and the directory structure it inherited (like `C:\Program Files\`, `C:\Windows\System32\`) remain to this day.

**The UNIX/Linux lineage** is fundamentally different. UNIX, developed at Bell Labs in the late 1960s, designed its file system around a single *tree* rooted at `/` (called "the root"). Everything — hard drives, removable media, network shares, even hardware devices — appears somewhere in this single tree. There are no drive letters; instead, a disk partition or USB stick gets "mounted" at a directory within the tree, and from that point on, it's accessed as if it were just a subdirectory. This unified namespace is more elegant and flexible for multi-user, networked environments.

Linux follows UNIX traditions. A few practical consequences flow from this directly:

- Paths use **forward slashes**: `/home/shaswat/documents/thesis.pdf`
- **Filenames are case-sensitive**: `File.txt`, `file.txt`, and `FILE.TXT` are three distinct files
- **Hidden files start with a dot**: `.bashrc`, `.config/` — any file or directory beginning with `.` is hidden from normal directory listings (use `ls -a` to reveal them)

> **What is POSIX?** POSIX (Portable Operating System Interface) is a family of standards defined by the IEEE that specifies how operating systems should behave — what system calls exist, how the shell works, what utilities are available, and how the file system should be structured. Being "POSIX-compliant" means a system follows these rules, making software portable across UNIX-like systems. Linux is largely POSIX-compliant, which is why shell scripts and programs written on macOS often work on Linux with little or no modification.

---

### The File Hierarchy Standard (FHS)

The **File Hierarchy Standard (FHS)**, maintained by the Linux Foundation, is the document that formally defines what belongs where in a Linux file system. It specifies the minimum set of directories and files every Linux system must have, describes the *purpose* of each directory, and establishes naming conventions. Software authors use this to know where to install their files; users and administrators use it to know where to look for things.

The latest version, FHS 3.0, is available at: https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html

---

### Two Key Axes

The FHS organizes the file system along two independent axes. Understanding these makes the overall structure click into place.

**Axis 1: Shareable vs. Non-Shareable**

A *shareable* file is one that can safely be stored on one machine and accessed by another over a network. Your home directory contents (`/home`) are shareable — a user's files are the same whether accessed from their laptop or from a server. In contrast, *device lock files* (files that record "this device is currently in use") are non-shareable, because they reflect the local state of a specific machine's hardware and would be meaningless or harmful if another machine tried to read them.

**Axis 2: Static vs. Variable**

A *static* file is one that does not change unless a system administrator explicitly modifies it — binaries, libraries, documentation. A *variable* file changes during normal operation — log files grow, mail spools fill up, databases update. This distinction matters for two practical reasons: (1) static files can be on a read-only filesystem for security, and (2) variable files need more frequent backup.

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

Your home directory contains both *visible* files and folders (Documents, Downloads, Desktop, Pictures, etc.) and *hidden* configuration files and directories. The hidden ones — identified by a leading dot — are created automatically by applications the first time they run, and store per-user settings so that configuration is isolated between users on the same machine.

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

`/root` is the home directory of the `root` user — the system's superuser (administrator). It exists at `/root` rather than `/home/root` for a deliberate reason: the `/home` directory might be on a separate disk partition. If that partition fails to mount during boot, the root user would be locked out of their own home directory, which would be catastrophic during system recovery. By keeping `/root` on the root partition itself, the superuser always has access.

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

> **A note on security:** Even if you have `sudo` access, the best practice is to avoid working as root directly. Use `sudo` for individual commands that require it. This limits the blast radius of mistakes — a mistyped `rm -rf` as root can destroy the entire system, while the same mistake as a normal user only destroys your home directory.

---

## 4. Executables & Commands

### `/bin` — Essential User Binaries

> **What is a binary?** "Binary" is shorthand for a compiled, executable program — as opposed to a script, which is plain text. When you write a C program and compile it, the output is a binary: a file containing machine code that the processor can execute directly.

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

`/sbin` holds binaries intended for system administration tasks — commands that typically require root privileges and are not needed for routine user activity. Think of it as the administrator's toolkit.

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

`/usr` is the largest directory on most Linux systems and the most layered in terms of structure. The name originally stood for "Unix System Resources" (not "user" as one might guess). It holds *non-essential* software — everything the system needs to run (kernel, shell, basic utilities) is in `/bin`, `/sbin`, and `/lib`. Everything else — the applications you actually use — lives in `/usr`.

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
> When a program is compiled, it uses code from external libraries — reusable collections of functions. There are two ways to include that code: (1) *static linking*, where the library code is copied directly into the executable at compile time, making the binary self-contained but large; and (2) *dynamic linking*, where the binary is compiled with only a reference to the library, and the actual code is loaded at runtime from a shared file.
>
> On Linux, these shared files are called **shared objects** and have the extension `.so` (e.g., `libc.so.6`). On Windows, the equivalent is a **DLL** (Dynamic Link Library), files with the `.dll` extension. The concept is identical — a single file containing compiled library code that multiple programs can share simultaneously in memory, reducing both disk usage and RAM consumption.
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

`/proc` (the "procfs" filesystem) is a virtual filesystem, like `/sys`, that exposes information about running processes and the kernel. Each running process gets a directory named after its **PID** (Process ID) — a unique integer assigned by the kernel to identify it. Inside that directory are files that describe everything about the process: memory usage, open file descriptors, the command that launched it, environment variables, and more.

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

One of UNIX's most elegant ideas is that *everything is a file*, including hardware devices. `/dev` contains **device files** — special files that represent physical or virtual hardware. When a program reads from or writes to one of these files, the kernel translates that operation into the appropriate hardware interaction.

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

`/etc` is the nerve center for system-wide configuration. The name is a remnant from early UNIX — it literally meant "et cetera," a catch-all for files that didn't fit elsewhere — but it has long since become the dedicated home for editable configuration files. The key word is *system-wide*: files here affect all users and all services. User-specific configuration lives in `~/.config/`.

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

`/mnt` is a generic, manually managed mount point. You use it when you want to attach something yourself: an additional hard drive, a network share, a disk image, or a partition you're working on. Unlike `/media`, nothing auto-mounts here — you're in full control.

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