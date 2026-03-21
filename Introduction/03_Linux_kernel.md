# Linux: Concepts, Mechanisms, and Recipes

## 0. Boot Sequence

Understanding where each component fits requires seeing the full boot sequence:

```
Power on
  └── UEFI/BIOS firmware
        └── finds and runs GRUB (bootloader)
              └── loads Linux kernel into RAM, runs it
                    └── kernel initialises hardware, mounts root filesystem
                          └── starts systemd (PID 1)
                                └── systemd starts all other services
                                      └── login prompt / desktop environment
```

Everything above "kernel" is firmware. Everything below "kernel" is userspace. The kernel is the bridge.

---

## 1. Firmware: UEFI

The first C code (and some assembly) that is executed by the CPU when the machine powers on. Lives in flash memory on the motherboard, with a fixed address the CPU executes by setting up stack in the CPU cache since RAM is not initialized yet. Its obviously not a part of the OS, but bridges the gap between powere on and unconnected live hardware to an integrated computer that can allows OS to run.

1. Power-On Self Test (POST) — During secure boot, it checks its own cryptograhic checksums, loads itself, verifies that RAM, CPU, and storage are functional.
2. Loads Device Drivers.
3. Reads the UEFI boot entry list (stored in NVRAM) to find where the bootloader lives on disk.
4. Loads and hands control to the bootloader.


```bash
# Inspect UEFI boot entries from within Linux
efibootmgr -v

# Check if system booted in UEFI mode
ls /sys/firmware/efi   # exists → UEFI; missing → legacy BIOS
```

---

## 2. Bootloader: GRUB
GRand Unified Bootloader. A small executable binary stored in the EFI System Partition of disk (or the MBR for legacy BIOS) and is loaded onto RAM by UEFI. Its sole job is to load the kernel, aftter UEFI has init RAM and storage

1. Presents a boot menu (multiple kernels or OSes) with a timer for default selection.
2. Loads the selected kernel binary (`vmlinuz`) and initial RAM disk (`initrd`/`initramfs`) into RAM.
3. Passes kernel command-line parameters (e.g., `root=/dev/sda2 quiet splash`).
4. Transfers control to the kernel entry point by passing infor about (memory address where kernel was loaded from, command line strings, UEFI memory map and other platfrom information from UEFI to kernel). GRUB is now done and the kernel kills it.

**`initramfs`:** A temporary minimal filesystem bundled alongside the kernel. The kernel mounts it in RAM first, uses it to load the drivers needed to access the real root filesystem (e.g., LUKS encryption, LVM, RAID drivers), then pivots to the real root and discards `initramfs`.

```bash
# GRUB config lives here — edit this, not /boot/grub/grub.cfg directly
/etc/default/grub
sudo update-grub           # regenerates /boot/grub/grub.cfg from the above

# View current kernel command line
cat /proc/cmdline

# See which kernel you are running
uname -r                   # e.g., 6.8.0-45-generic
```

The key insight is that GRUB is a specialised file reader with a boot menu. Its core competency is understanding filesystems (ext4, xfs, LUKS, LVM, btrfs) without the OS, finding the right files, loading them into the right memory addresses, and handing the CPU to the kernel with a precisely specified set of information.

---

## 3. The Linux Kernel

A body of C code (with architecture-specific assembly) that runs in privileged mode (Ring 0 on x86-64). It is the one program that has unrestricted access to all hardware. Everything else runs in unprivileged user mode (Ring 3) and can only access hardware by asking the kernel.

**Core subsystems:**

| Subsystem | Responsibility |
|---|---|
| **Process / Thread Scheduler** | Decides which task runs on which CPU core and for how long |
| **Memory Management** | Virtual address spaces, page tables, page fault handling, `malloc` backing |
| **Virtual File System (VFS)** | Unified `open/read/write/close` interface over any filesystem (ext4, xfs, tmpfs, procfs...) |
| **Device Drivers** | Code that speaks the hardware protocol for each device (NIC, disk, GPU, keyboard) |
| **Networking Stack** | TCP/IP implementation, socket layer, packet routing |
| **IPC Mechanisms** | Pipes, signals, shared memory, sockets, message queues |
| **Interrupt & Exception Handlers** | Responds to hardware interrupts (timer, NIC, disk) and CPU exceptions (page fault, divide-by-zero) |
| **Security & Access Control** | Permissions, capabilities, namespaces, seccomp, LSMs (SELinux, AppArmor) |

**Kernel vs. OS:** The kernel is the core. The operating system is the kernel *plus* all the userspace software (shell, init, libraries, daemons) that makes the system usable. Linux is a kernel. Ubuntu/Fedora/Arch are operating systems built around it.

```bash
# Kernel version
uname -r

# All kernel messages since boot (hardware detection, driver loading, errors)
dmesg | less
dmesg | grep -i error

# Kernel configuration options for the running kernel
zcat /proc/config.gz | grep CONFIG_BPF   # if available

# Loaded kernel modules (drivers loaded at runtime)
lsmod
modinfo <module_name>      # details about a module
sudo modprobe <module>     # load a module
sudo rmmod <module>        # unload a module
```

---

## 4. System Call Interface

The controlled gateway coordinating between user mode and kernel mode. The only way a user-space program can ask the kernel to do something.

**Mechanism (x86-64):**
1. User program places syscall number in `RAX`, arguments in `RDI, RSI, RDX, R10, R8, R9`.
2. Executes the `syscall` instruction — CPU atomically switches to Ring 0 and jumps to the kernel's syscall handler.
3. Kernel validates permissions and executes the request.
4. Returns result in `RAX`; `sysretq` switches back to Ring 3.

 The C standard library (glibc) wraps every syscall in a normal C function. When a programmer calls `open()`, `read()`, `fork()`, or `malloc()` in a C program, they are calling glibc wrappers that invoke the corresponding syscall.

**Syscall categories:**

| Category | Examples |
|---|---|
| Process control | `fork`, `execve`, `exit`, `waitpid`, `clone` |
| Memory | `mmap`, `munmap`, `brk` |
| File I/O | `open`, `read`, `write`, `close`, `stat`, `lseek` |
| Directory | `mkdir`, `rmdir`, `opendir`, `getdents` |
| Networking | `socket`, `bind`, `connect`, `accept`, `send`, `recv` |
| IPC | `pipe`, `msgget`, `shmget`, `semget` |
| Signals | `kill`, `sigaction`, `sigprocmask` |
| Info | `getpid`, `gettid`, `getuid`, `uname`, `clock_gettime` |

```bash
# Trace all syscalls made by a program
strace ls
strace -e trace=openat,read,write cat /etc/hostname

# Count syscalls by type
strace -c ls /

# See what syscalls a specific process is making right now
strace -p <PID>

# Which library provides a function
man 2 open   # section 2 = syscalls
man 3 fopen  # section 3 = library functions (glibc wrappers)
```

---

## 5. Virtual File System (VFS)

 A kernel abstraction layer that presents a single unified tree of files regardless of which physical filesystem or device they reside on.

 From user space, `open("/etc/hostname")`, `open("/proc/1/status")`, and `open("/dev/sda")` all use the exact same syscall. The VFS dispatches each call to the appropriate driver — ext4 driver, procfs driver, block device driver — transparently. User programs never need to know which filesystem they are talking to.

**Everything is a file:** In Linux, following UNIX philosophy, the VFS abstraction is used for:
- Regular files and directories (ext4, xfs, btrfs on disk)
- `/proc` — a virtual filesystem exposing kernel and process state as files
- `/sys` — exposes device and driver state
- `/dev` — device files (block devices like `/dev/sda`, character devices like `/dev/tty`)
- Sockets and pipes (accessible via file descriptors)

**Inode:** The fundamental data structure representing a file or directory inside a filesystem. An inode stores metadata — file size, owner, permissions, timestamps, and pointers to the actual data blocks on disk. The filename is *not* stored in the inode; it lives in the directory entry that references the inode. This separation enables hard links: multiple filenames pointing to the same inode (same data).

```bash
# Show inode number of files
ls -i /etc/hostname
stat /etc/hostname       # full inode metadata

# Hard link: two names, one inode
ln file1 file2
ls -i file1 file2        # same inode number

# Soft (symbolic) link: a file whose content is a path string
ln -s /etc/hostname mylink
ls -la mylink

# Filesystem usage
df -h                    # disk usage per mounted filesystem
df -i                    # inode usage (can run out of inodes before disk space)
du -sh /var/log          # space used by a directory

# Mount and unmount filesystems
lsblk                    # list block devices and mount points
mount | grep sda         # see what's mounted
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb

# Check/repair filesystem (must be unmounted)
sudo fsck /dev/sdb1
```

### The Linux Filesystem Hierarchy

The Filesystem Hierarchy Standard (FHS) defines where things live:

| Path | Contents |
|---|---|
| `/` | Root of the entire tree |
| `/bin`, `/usr/bin` | User command binaries (`ls`, `cat`, `grep`) |
| `/sbin`, `/usr/sbin` | System administration binaries (`ip`, `mount`, `fdisk`) |
| `/lib`, `/usr/lib` | Shared libraries (`.so` files) |
| `/etc` | System-wide configuration files (text, editable) |
| `/home/user` | User's home directory |
| `/root` | Root user's home directory |
| `/var` | Variable data: logs (`/var/log`), spools, caches |
| `/tmp` | Temporary files (cleared on reboot) |
| `/proc` | Virtual: live kernel and process state |
| `/sys` | Virtual: device and driver attributes |
| `/dev` | Virtual: device files |
| `/boot` | Kernel, initramfs, GRUB files |
| `/opt` | Optional third-party software |
| `/run` | Runtime data (PIDs, sockets) — tmpfs, cleared on reboot |
| `/usr` | Secondary hierarchy: read-only user data and programs |

```bash
# Explore /proc — everything is a live kernel read
cat /proc/cpuinfo           # CPU details
cat /proc/meminfo           # RAM usage
cat /proc/uptime            # seconds since boot
cat /proc/loadavg           # 1/5/15-min load averages
ls /proc/<PID>/             # everything about a specific process
cat /proc/<PID>/status      # state, memory, threads
cat /proc/<PID>/maps        # virtual memory map (address space layout)
ls -la /proc/<PID>/fd/      # open file descriptors
cat /proc/<PID>/cmdline     # how the process was invoked
```

---

## 6. File Descriptors

 Non-negative integers that represent an open resource in a process. Every open file, socket, pipe, or device in a process is referenced by a file descriptor. The kernel maintains a per-process file descriptor table mapping each integer to a kernel file object.

**Standard descriptors (inherited by every process):**

| FD | Name | Default destination |
|---|---|---|
| 0 | stdin | keyboard (terminal) |
| 1 | stdout | terminal |
| 2 | stderr | terminal |

When a process calls `open()`, it gets the lowest available integer ≥ 3.

**Inheritance:** Child processes created with `fork()` inherit copies of the parent's file descriptor table. This is how shell pipelines work — the shell creates a pipe, forks a child, and the child inherits the read and write ends of the pipe.

```bash
# Redirecting file descriptors in the shell
command > file          # redirect stdout (FD 1) to file
command 2> file         # redirect stderr (FD 2) to file
command > file 2>&1     # redirect stdout to file, then redirect stderr to same place as stdout
command &> file         # shorthand for the above (bash)
command < file          # redirect stdin from file
command 2>/dev/null     # discard stderr

# Pipe: connects stdout of left to stdin of right
ls | grep ".conf"

# View a process's open FDs
ls -la /proc/<PID>/fd/
lsof -p <PID>           # list open files for a process
lsof /var/log/syslog    # which processes have this file open

# Which process is using a port
lsof -i :8080
lsof -i tcp
```

---

## 7. Signals

**What they are:** Asynchronous notifications sent to a process by the kernel, another process, or the process itself. They interrupt the process's normal execution flow. A signal is identified by a number and a name.

**How delivery works:** The kernel sets a pending signal flag in the process's `task_struct`. Before the process returns to user mode (after a syscall or interrupt), the kernel checks for pending signals and delivers them — jumping to the registered signal handler, or applying a default action.

**Common signals:**

| Signal | Number | Default action | Meaning / Common use |
|---|---|---|---|
| `SIGHUP` | 1 | Terminate | Hangup. Sent when terminal closes. Many daemons reload config on SIGHUP. |
| `SIGINT` | 2 | Terminate | Interrupt from keyboard (`Ctrl+C`) |
| `SIGQUIT` | 3 | Core dump | Quit from keyboard (`Ctrl+\`) |
| `SIGKILL` | 9 | Terminate | Unconditional kill — **cannot be caught, blocked, or ignored** |
| `SIGTERM` | 15 | Terminate | Polite termination request — **can be caught** (allows cleanup) |
| `SIGSTOP` | 19 | Stop | Pause process — **cannot be caught or ignored** |
| `SIGCONT` | 18 | Continue | Resume a stopped process |
| `SIGCHLD` | 17 | Ignore | Child process changed state (used by parent to `wait()`) |
| `SIGSEGV` | 11 | Core dump | Segmentation fault (invalid memory access) |
| `SIGALRM` | 14 | Terminate | Timer expired (`alarm()` syscall) |
| `SIGUSR1/2` | 10/12 | Terminate | User-defined — application-specific use |

**`SIGKILL` vs. `SIGTERM`:** Always try `SIGTERM` first. It gives the process a chance to flush buffers, release locks, and clean up. `SIGKILL` tears the process down at the kernel level with no warning — this can leave files partially written, sockets in bad state, or database transactions incomplete.

```bash
# Send signals to processes
kill <PID>             # sends SIGTERM by default
kill -9 <PID>          # sends SIGKILL (last resort)
kill -SIGTERM <PID>    # explicit signal name
kill -HUP <PID>        # send SIGHUP (often reloads config)
killall nginx          # send SIGTERM to all processes named nginx
pkill -f "python app"  # match by full command line

# Keyboard shortcuts in terminal
Ctrl+C                 # SIGINT  — interrupt running command
Ctrl+Z                 # SIGTSTP — stop (pause) running command
fg                     # resume stopped job in foreground
bg                     # resume stopped job in background

# View pending/blocked signals
cat /proc/<PID>/status | grep Sig
```

---

## 8. Processes and the `/proc` Filesystem

**`/proc`** is a virtual filesystem — it has no backing storage on disk. The kernel generates its contents on-the-fly in response to reads. It is the primary window into live kernel and process state.

```bash
# Process-level inspection
cat /proc/<PID>/status       # name, state, PID, PPID, threads, memory usage
cat /proc/<PID>/maps         # virtual address space: each mapped region, permissions, backing file
cat /proc/<PID>/smaps        # per-region memory breakdown (RSS, PSS, anonymous vs file-backed)
cat /proc/<PID>/cmdline      # null-separated argv (the command that launched it)
cat /proc/<PID>/environ      # null-separated environment variables
cat /proc/<PID>/exe          # symlink to the executable binary
ls -la /proc/<PID>/fd/       # open file descriptors (symlinks to what each FD points to)
cat /proc/<PID>/io           # bytes read/written
cat /proc/<PID>/net/tcp      # open TCP connections for this process's namespace

# System-level
cat /proc/version            # kernel version
cat /proc/cpuinfo            # per-core CPU info (model, MHz, cache, flags)
cat /proc/meminfo            # RAM: total, free, available, buffers, cached, swap
cat /proc/loadavg            # 1/5/15-min load averages + running/total threads
cat /proc/net/dev             # network interface statistics
cat /proc/sys/               # tunable kernel parameters (sysctl)

# Process tree
pstree -p                    # visual tree with PIDs
ps aux                       # all processes: user, PID, CPU%, MEM%, command
ps aux --forest              # show parent-child relationships
ps -eo pid,ppid,stat,comm    # custom columns

# Real-time process monitoring
top                          # interactive, sorted by CPU usage
htop                         # better UI, mouse-friendly
```

---

## 9. Zombie Processes

A process that has finished execution (`exit()`) but whose PCB (`task_struct`) has not yet been removed from the kernel's process table.

**Why they exist:** When a child process exits, the kernel does not immediately destroy its PCB. It retains the exit status code so the parent can retrieve it. The process is "dead" — it executes no instructions, holds no memory — but its PCB slot remains occupied with state `Z` (zombie). The parent must call `wait()` or `waitpid()` to collect this exit status, after which the kernel finally destroys the PCB. This collection step is called **reaping**.

**Why they are a problem:** A small number of zombies is harmless. But if a buggy parent creates thousands of children and never calls `wait()`, zombie PCBs accumulate. Since PIDs are a finite namespace (typically 0–4194304 on Linux), this can exhaust all available PIDs, preventing any new process or thread from being created system-wide.

**Orphan processes:** If a parent exits *before* its children, the children become orphans. The kernel automatically re-parents all orphans to `systemd` (PID 1), which calls `wait()` periodically — so orphans are properly reaped.

```bash
# Identify zombies
ps aux | grep 'Z'
ps aux | awk '$8 == "Z"'   # state column is Z

# Zombie count
cat /proc/loadavg          # format: 0.10 0.15 0.20 1/312 9876
                           # "1/312" means 1 running out of 312 total tasks

# Find the parent of a zombie and tell it to clean up
# (SIGCHLD prompts the parent to call wait())
kill -SIGCHLD <parent_PID>

# If the parent itself is stuck and unkillable, reboot is the only fix
# (unless you can kill the parent — orphaned zombies get re-parented to systemd which reaps them)
```

---

## 10. Namespaces

**What they are:** A kernel mechanism that partitions global resources so that processes in different namespaces see different views of those resources. Each process belongs to exactly one namespace of each type. Namespaces are the isolation primitive underlying containers (Docker, Podman, LXC).

**Types:**

| Namespace | Flag | Isolates |
|---|---|---|
| **PID** | `CLONE_NEWPID` | Process ID space. A process in a new PID namespace sees itself as PID 1. PIDs inside do not conflict with outside. |
| **Network** | `CLONE_NEWNET` | Network interfaces, IP addresses, routing tables, iptables rules, ports. A container gets its own `eth0`. |
| **Mount** | `CLONE_NEWNS` | Mount table. Changes to mounts (mounting, unmounting) are invisible outside the namespace. |
| **UTS** | `CLONE_NEWUTS` | Hostname and domain name. A container can have its own hostname. |
| **IPC** | `CLONE_NEWIPC` | System V IPC (shared memory, semaphores, message queues) and POSIX message queues. |
| **User** | `CLONE_NEWUSER` | User and group ID mappings. A process can be UID 0 (root) inside a namespace but an unprivileged UID outside. |
| **Time** | `CLONE_NEWTIME` | System clock offsets (useful for testing time-sensitive code). |
| **Cgroup** | `CLONE_NEWCGROUP` | The cgroup root visible to the process. |

```bash
# See which namespaces a process belongs to
ls -la /proc/<PID>/ns/

# List all namespaces on the system
lsns

# Run a command in a new network namespace (isolated network stack)
sudo unshare --net bash
ip link   # only sees lo — no real network interfaces

# Enter the namespaces of a running container/process
sudo nsenter -t <PID> --net --pid bash

# Inspect namespace of a running Docker container
docker inspect <container> | grep -i pid
sudo nsenter -t <PID> --net -- ip addr
```

---

## 11. Cgroups (Control Groups)

**What they are:** A kernel mechanism for organising processes into hierarchical groups and applying resource limits, priorities, and accounting to each group. If namespaces control what a process *can see*, cgroups control what a process *can use*.

**What can be controlled:**

| Controller | What it limits / tracks |
|---|---|
| `cpu` | CPU time allocation (shares, quotas, periods) |
| `cpuset` | Which specific CPU cores and NUMA nodes a group can use |
| `memory` | RAM limit, swap limit, OOM kill behaviour |
| `io` | Disk read/write bandwidth and IOPS |
| `pids` | Maximum number of processes/threads in the group |
| `net_cls`, `net_prio` | Network traffic classification and priority |
| `freezer` | Suspend/resume all processes in a group atomically |

**cgroups v1 vs. v2:** Linux now defaults to cgroups v2, which unifies the hierarchy (one tree for all controllers) and is cleaner to reason about. Docker, systemd, and Kubernetes all use cgroups v2 on modern systems.

```bash
# The cgroup filesystem
ls /sys/fs/cgroup/          # cgroups v2: all controllers in one unified tree

# See which cgroup a process belongs to
cat /proc/<PID>/cgroup

# systemd automatically creates a cgroup per service and per user session
systemctl status nginx      # shows cgroup path under "CGroup:"
systemd-cgls                # visual tree of all cgroups

# Inspect resource usage of a cgroup
cat /sys/fs/cgroup/system.slice/memory.current     # current memory usage in bytes
cat /sys/fs/cgroup/system.slice/cpu.stat           # CPU usage counters

# Set a memory limit on a service
sudo systemctl set-property nginx.service MemoryMax=512M
sudo systemctl set-property nginx.service CPUQuota=50%

# Run a command with resource limits (uses cgroups under the hood)
systemd-run --scope -p MemoryMax=256M -p CPUQuota=25% ./my_program
```

---

## 12. `systemd` and Services

**What it is:** The first userspace process started by the kernel (PID 1). It is the init system, service manager, and more. All other processes are descendants of systemd.

**Key roles:**
- **Parallel service startup** at boot (dependencies expressed declaratively, not procedurally)
- **Service lifecycle management** (start, stop, restart, status)
- **Dependency ordering** (start network before services that need it)
- **Socket activation** (service only starts when a connection arrives on its socket)
- **Logging** via `journald` (structured, indexed, binary journal)
- **Process supervision** (auto-restart crashed services)
- **Orphan reaping** (all orphaned processes are re-parented to PID 1)

**Unit files** describe services, sockets, timers, and mounts. Located in `/lib/systemd/system/` (distro), `/etc/systemd/system/` (local overrides, takes precedence).

```bash
# Service management
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx        # reload config without full restart (if supported)
sudo systemctl status nginx
sudo systemctl enable nginx        # start on boot
sudo systemctl disable nginx       # do not start on boot
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx

# View all units
systemctl list-units --type=service
systemctl list-units --failed

# Inspect a unit file
systemctl cat nginx
systemctl show nginx               # all properties with values

# Override a unit without editing the original
sudo systemctl edit nginx          # creates /etc/systemd/system/nginx.service.d/override.conf

# Journal (logs)
journalctl                         # all logs, oldest first
journalctl -u nginx                # logs for a specific service
journalctl -f                      # follow (like tail -f)
journalctl -f -u nginx             # follow a service's logs
journalctl --since "1 hour ago"
journalctl -p err                  # only error priority and above
journalctl -b                      # only since last boot
journalctl -b -1                   # logs from the previous boot
journalctl --disk-usage

# Targets (like runlevels)
systemctl get-default              # default boot target (usually graphical.target or multi-user.target)
sudo systemctl isolate rescue.target  # switch to rescue mode
```

**Minimal service unit file example:**
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target        # start after network is up

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/server --port 8080
Restart=on-failure          # restart if it crashes
RestartSec=5

[Install]
WantedBy=multi-user.target  # start in multi-user mode
```

---

## 13. Users, Permissions, and Root

### Users and Groups

Every process runs as a user (UID) and a group (GID). The kernel enforces all access control decisions based on these.

| Concept | Detail |
|---|---|
| **UID 0 (root)** | Superuser. Bypasses almost all permission checks. Has unrestricted access to the entire system. |
| **Regular users** | UID ≥ 1000 (convention). Constrained to their own files and explicitly granted permissions. |
| **Groups** | A process has one primary GID and zero or more supplementary GIDs. Used for shared access. |
| **`/etc/passwd`** | Maps username → UID, GID, home dir, shell |
| **`/etc/shadow`** | Hashed passwords (readable only by root) |
| **`/etc/group`** | Maps group name → GID → member list |

### File Permissions

Every file has an owner (user), a group, and a 9-bit permission field:

```
-rwxr-xr--  1  alice  dev  4096  Mar 18  file.sh
│└┬┘└┬┘└┬┘
│ │  │  └─ others:  r-- (read only)
│ │  └──── group:   r-x (read + execute)
│ └─────── owner:   rwx (read + write + execute)
└───────── type: - = regular file, d = directory, l = symlink, c = char device
```

**Numeric (octal) representation:** r=4, w=2, x=1. Each group of three bits sums to a digit.
- `755` = rwxr-xr-x (owner full, group/others read+execute) — typical for executables and directories
- `644` = rw-r--r-- (owner read+write, group/others read) — typical for config files
- `600` = rw------- — private files (SSH keys)

**Special bits:**
- **setuid** (on executable): process runs as the *file owner*, not the caller. Example: `/usr/bin/passwd` is owned by root with setuid, so any user can run it and it gets root privileges to modify `/etc/shadow`.
- **setgid** (on directory): new files created inside inherit the directory's group.
- **sticky bit** (on directory): users can only delete their own files, even if they have write permission on the directory. Used on `/tmp`.

```bash
# View permissions
ls -la
stat file

# Change permissions
chmod 755 script.sh
chmod +x script.sh       # add execute for all
chmod u+w,g-w file       # symbolic: add write for user, remove for group
chmod -R 644 /var/www    # recursive

# Change owner and group
chown alice file
chown alice:dev file
chown -R www-data:www-data /var/www

# Numeric identity of current user
id                       # uid=1000(alice) gid=1000(alice) groups=...
whoami

# Run as root
sudo command             # run as root (logged, requires password)
sudo -i                  # interactive root shell
su - alice               # switch to user alice (requires alice's password)

# Sudoers configuration
sudo visudo              # safe editor for /etc/sudoers
# Allow alice to run any command as root without password:
# alice ALL=(ALL) NOPASSWD: ALL
```

---

## 14. Package Management

**What it is:** A system for installing, updating, removing, and tracking software. Packages are archives containing binaries, libraries, config files, and metadata (dependencies, version, maintainer). The package manager resolves dependencies and manages upgrades atomically.

### APT (Debian / Ubuntu)

```bash
sudo apt update                     # refresh package index from repositories
sudo apt upgrade                    # upgrade all installed packages
sudo apt full-upgrade               # upgrade + handle dependency changes (may remove packages)

sudo apt install nginx              # install
sudo apt install nginx=1.24.0       # specific version
sudo apt remove nginx               # remove (keep config files)
sudo apt purge nginx                # remove + delete config files
sudo apt autoremove                 # remove orphaned dependencies

apt search nginx                    # search package name/description
apt show nginx                      # details: version, deps, description
apt list --installed                # all installed packages
apt list --upgradable

# Repository configuration
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
```

### DNF / YUM (Fedora / RHEL / CentOS)

```bash
sudo dnf install nginx
sudo dnf remove nginx
sudo dnf update
sudo dnf search nginx
sudo dnf info nginx
sudo dnf list installed
```

### Pacman (Arch Linux)

```bash
sudo pacman -Syu                    # sync + full system upgrade
sudo pacman -S nginx                # install
sudo pacman -R nginx                # remove
sudo pacman -Ss nginx               # search
sudo pacman -Qi nginx               # info on installed package
sudo pacman -Ql nginx               # list files installed by package
```

---

## 15. Processes: Practical Tools

```bash
# Snapshot of all processes
ps aux                   # BSD style: all processes, user, CPU%, MEM%
ps -ef                   # UNIX style: all processes, PPID visible
ps -eo pid,ppid,user,stat,pcpu,pmem,comm   # custom columns

# Process tree
pstree -p                # tree with PIDs
pstree -p -u             # tree with PIDs and usernames

# Find a process
pgrep nginx              # PIDs matching name
pgrep -a nginx           # PIDs + full command line
pidof nginx              # all PIDs for exact name

# Real-time monitoring
top                      # press P (CPU sort), M (mem sort), k (kill), q (quit)
htop                     # better interactive monitor, mouse support

# Manage background jobs in the shell
command &                # run in background
jobs                     # list background jobs
fg %1                    # bring job 1 to foreground
bg %1                    # resume job 1 in background
nohup command &          # run immune to SIGHUP (survives terminal close)
disown %1                # remove job from shell's job table (survives shell exit)

# Adjust priority
nice -n 10 command       # start with lower priority (nice value 10, range -20 to 19)
sudo nice -n -5 command  # higher priority (requires root for negative nice)
renice -n 5 -p <PID>     # change priority of running process
```

---

## 16. Networking Stack and Tools

**What it is:** The kernel implements the TCP/IP stack — routing, TCP connection management, UDP, ICMP. User programs interact through sockets (a socket is a file descriptor representing a network endpoint).

```bash
# Interface management (modern: ip; legacy: ifconfig)
ip addr                  # show all interfaces and IP addresses
ip addr add 192.168.1.10/24 dev eth0
ip addr del 192.168.1.10/24 dev eth0
ip link set eth0 up/down
ip link show

# Routing
ip route                 # show routing table
ip route add default via 192.168.1.1   # set default gateway
ip route add 10.0.0.0/8 via 10.0.0.1 dev eth0

# DNS resolution
cat /etc/resolv.conf     # configured DNS servers
cat /etc/hosts           # static hostname→IP mappings (checked before DNS)
dig example.com          # DNS lookup with full response
dig @8.8.8.8 example.com # query a specific DNS server
nslookup example.com
host example.com

# Connectivity testing
ping -c 4 8.8.8.8
traceroute 8.8.8.8       # or tracepath
mtr 8.8.8.8              # continuous traceroute (better for diagnostics)
curl -v https://example.com
wget https://example.com/file.tar.gz

# Sockets and ports
ss -tlnp                 # TCP listening sockets with process
ss -tunp                 # TCP+UDP, numeric, with processes
ss -s                    # socket statistics summary
lsof -i :443             # which process owns port 443
lsof -i tcp              # all TCP sockets

# Capture packets
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 host 10.0.0.1 -w capture.pcap   # save to file

# NetworkManager (desktop/server)
nmcli device status
nmcli connection show
nmcli connection up "My WiFi"
```

---

## 17. SSH

**What it is:** Secure Shell. An encrypted protocol for remote login and command execution. The client (`ssh`) connects to a server daemon (`sshd`) on port 22 (default). Authentication is via password or public-key cryptography.

**Public-key authentication:** You generate a keypair (private key stays on your machine, public key goes on the server in `~/.ssh/authorized_keys`). The server challenges the client to prove it has the private key without sending it — more secure and more convenient than passwords.

```bash
# Generate a keypair
ssh-keygen -t ed25519 -C "your@email.com"
# Creates ~/.ssh/id_ed25519 (private — keep secret) and ~/.ssh/id_ed25519.pub (public)

# Copy public key to a server
ssh-copy-id user@server          # appends pubkey to server's authorized_keys
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Connect
ssh user@server
ssh user@server -p 2222          # non-default port
ssh -i ~/.ssh/other_key user@server   # specify key

# Run a single command remotely
ssh user@server "df -h && uptime"

# SSH config file (~/.ssh/config) — define aliases
# Host myserver
#   HostName 203.0.113.10
#   User alice
#   Port 2222
#   IdentityFile ~/.ssh/id_ed25519
ssh myserver                     # uses all settings from config

# File transfer
scp file.txt user@server:/home/user/
scp user@server:/var/log/app.log ./
scp -r ./dir user@server:~/      # recursive directory copy

# rsync — efficient sync (only transfers diffs)
rsync -avz ./local/ user@server:/remote/    # sync local → remote
rsync -avz user@server:/remote/ ./local/    # sync remote → local
rsync -avz --delete ./local/ user@server:/remote/  # delete files on remote not in local
rsync -n ./local/ user@server:/remote/     # dry run: show what would change

# SSH tunnels
ssh -L 8080:localhost:80 user@server   # local port 8080 → server's port 80
ssh -R 9090:localhost:3000 user@server # server's port 9090 → your port 3000
ssh -D 1080 user@server               # SOCKS5 proxy through server

# Keep connections alive in ~/.ssh/config:
# ServerAliveInterval 60
# ServerAliveCountMax 3
```

---

## 18. Shell and the Terminal

**Terminal emulator:** A graphical application that renders text input/output and hosts a shell process. Examples: GNOME Terminal, Alacritty, iTerm2, Windows Terminal. The terminal is just an I/O surface.

**Shell:** A command interpreter that reads your input, parses it, and either executes built-in commands or `fork()`s + `exec()`s external programs. Common shells: `bash`, `zsh`, `fish`.

```bash
# Shell basics
echo $SHELL              # current shell
echo $PATH               # directories searched for commands
which python3            # full path of a command
type ls                  # whether it's a builtin, alias, or external
command -v ls            # POSIX portable version of which

# Variables
NAME="Alice"
echo $NAME
echo ${NAME:-default}    # use "default" if NAME is unset or empty
export NAME              # make visible to child processes
unset NAME

# Command substitution
DATE=$(date +%Y-%m-%d)
echo "Today is $DATE"

# Arithmetic
echo $((2 ** 10))        # 1024
x=5; echo $((x * 3))

# Useful shortcuts
Ctrl+A / Ctrl+E          # jump to start / end of line
Ctrl+W                   # delete word before cursor
Ctrl+U                   # delete to start of line
Ctrl+R                   # reverse search command history
Ctrl+L                   # clear screen
!!                       # repeat last command
!$                       # last argument of previous command
sudo !!                  # repeat last command with sudo

# History
history
history | grep ssh
!42                      # run command number 42 from history

# Aliases
alias ll='ls -alF'
alias gs='git status'
# Put in ~/.bashrc or ~/.zshrc for persistence
```

---

## 19. Daemons and Background Services

**What a daemon is:** A background process that runs continuously, not attached to any terminal. Daemons typically: start at boot, have a name ending in `d` (e.g., `sshd`, `crond`, `nginx`), log to `journald` or `/var/log`, and wait for events (network connections, timers, file changes).

**Lifecycle:** Modern daemons are managed by systemd (see Section 12). Legacy daemons forked themselves and wrote their PID to `/var/run/<name>.pid`.

### Cron Jobs

Cron is the traditional scheduler for recurring tasks. The `cron` daemon reads crontab files and runs commands on schedule.

**Crontab format:** `minute hour day-of-month month day-of-week command`

```
# ┌──────── minute (0-59)
# │  ┌───── hour (0-23)
# │  │  ┌── day of month (1-31)
# │  │  │  ┌─ month (1-12)
# │  │  │  │  ┌ day of week (0=Sun, 7=Sun)
# │  │  │  │  │
  0  2  *  *  * /opt/backup.sh          # run at 02:00 every day
  */5 * *  *  * /usr/bin/check.sh       # every 5 minutes
  0   9  *  *  1-5 /opt/report.sh       # 09:00 Mon-Fri
  0   0  1  *  * /opt/monthly.sh        # midnight on 1st of each month

```

```bash
crontab -e               # edit your crontab (uses $EDITOR)
crontab -l               # list your crontab
crontab -r               # remove your crontab (careful!)
sudo crontab -u alice -l # list crontab for user alice

# System-wide cron directories (scripts dropped here run automatically)
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/

# Cron logs
journalctl -u cron
grep CRON /var/log/syslog
```

**systemd timers** are the modern alternative — more precise, log to journald, can be transient:

```bash
# List all timers
systemctl list-timers --all

# Example timer unit: /etc/systemd/system/backup.timer
# [Timer]
# OnCalendar=daily
# OnBootSec=10min
# [Install]
# WantedBy=timers.target
```

---

## 20. Build Tools: `gcc` and `make`

### gcc — GNU Compiler Collection

**What it does:** Compiles C/C++ (and other languages) source code into executables, object files, or shared libraries. A "compilation" is actually a four-stage pipeline:

```
Source (.c)
  → Preprocessor (expand #include, #define) → Preprocessed source
  → Compiler (generate assembly)            → Assembly (.s)
  → Assembler (generate machine code)       → Object file (.o)
  → Linker (combine objects + libraries)    → Executable or .so
```

```bash
# Basic compilation
gcc hello.c -o hello         # compile and link → executable
./hello

# See each stage
gcc -E hello.c               # preprocess only (output to stdout)
gcc -S hello.c               # compile to assembly → hello.s
gcc -c hello.c               # compile to object file → hello.o
gcc hello.o -o hello         # link only

# Flags
gcc -Wall -Wextra hello.c -o hello    # enable warnings
gcc -O2 hello.c -o hello              # optimisation level 2
gcc -O0 -g hello.c -o hello           # no optimisation + debug symbols (for gdb)
gcc -o output src1.c src2.c           # compile multiple source files
gcc main.c -lm -o calc                # link with libm (math library)

# Shared libraries
gcc -fPIC -shared utils.c -o libutils.so   # create shared library
gcc main.c -L. -lutils -o main             # link against it
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH  # tell loader where to find it
ldd ./main                                  # list shared library dependencies

# Debugging
gdb ./hello                  # start debugger
gdb -p <PID>                 # attach to running process
valgrind ./hello             # memory error detector (leaks, overflows)
```

### make — Build Automation

**What it is:** A build system that tracks dependencies between files and re-runs only the commands needed to bring targets up to date. Reads a `Makefile`.

**Core concept:** A rule has a *target*, *prerequisites*, and a *recipe*. If the target file is older than any prerequisite, the recipe is re-run.

```makefile
# Makefile structure:
# target: prerequisites
# <TAB> recipe (must be a real tab, not spaces)

CC = gcc
CFLAGS = -Wall -O2

# Final executable depends on object files
main: main.o utils.o
	$(CC) $(CFLAGS) main.o utils.o -o main

# Object files depend on source + headers
main.o: main.c utils.h
	$(CC) $(CFLAGS) -c main.c

utils.o: utils.c utils.h
	$(CC) $(CFLAGS) -c utils.c

# Phony targets (not real files)
.PHONY: clean install

clean:
	rm -f *.o main

install:
	cp main /usr/local/bin/
```

```bash
make                 # build default target (first target in Makefile)
make clean           # run the clean target
make -j4             # parallel build with 4 jobs
make -n              # dry run: print commands without running them
make CFLAGS="-O0 -g" # override a variable from command line

# For projects using autotools:
./configure
make
sudo make install

# For cmake projects:
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

---

## 21. System Logs

All log data flows to `journald` (systemd's logging daemon) and/or to files in `/var/log/`.

```bash
# journalctl — the primary tool
journalctl                           # all logs
journalctl -f                        # follow live
journalctl -u nginx -f               # follow a service
journalctl -p err..alert             # severity: debug info notice warning err crit alert emerg
journalctl --since "2024-01-01" --until "2024-01-02"
journalctl --since "1 hour ago"
journalctl -b                        # current boot
journalctl -b -1                     # previous boot
journalctl -b --list-boots           # list all recorded boots
journalctl -k                        # kernel messages only (dmesg equivalent)
journalctl _PID=1234                 # logs from a specific PID
journalctl _UID=1000                 # logs from a specific user

# Traditional log files
/var/log/syslog       # general system log (Debian/Ubuntu)
/var/log/messages     # general system log (RHEL/Fedora)
/var/log/auth.log     # authentication events (sudo, ssh, su)
/var/log/kern.log     # kernel messages
/var/log/apt/         # apt package operations
/var/log/nginx/       # nginx access and error logs

# Real-time tailing
tail -f /var/log/nginx/error.log
tail -f /var/log/syslog | grep error
```

---

## Summary: The Linux Mental Model

```
Hardware
  └── UEFI firmware → GRUB bootloader
        └── Linux Kernel (Ring 0)
              ├── Process Scheduler        (who runs on which core)
              ├── Memory Management        (virtual address spaces, page tables)
              ├── VFS                      (unified file interface → ext4, proc, sys, dev)
              ├── Device Drivers           (hardware protocol per device)
              ├── Networking Stack         (TCP/IP, sockets)
              ├── IPC                      (pipes, signals, sockets, shared memory)
              └── System Call Interface    (controlled gate to all of the above)
                    │
              ┌─────┴─────────────────────────────────────────────────────────────┐
              │  Userspace (Ring 3)                                               │
              │  glibc (wraps syscalls)                                           │
              │    └── systemd (PID 1)                                            │
              │          ├── sshd, nginx, crond, ... (daemons / services)         │
              │          ├── login / display manager                              │
              │          └── user session                                         │
              │                └── shell (bash/zsh)                              │
              │                      └── your programs                           │
              └───────────────────────────────────────────────────────────────────┘

Key isolation boundaries:
  Ring 0 / Ring 3       hardware-enforced: user code cannot touch kernel memory
  Virtual address spaces separate per process, enforced by MMU + page tables
  Namespaces            isolate views (PIDs, network, mounts, users)
  Cgroups               limit resources (CPU, RAM, I/O, PIDs)
  File permissions       DAC: owner/group/other × read/write/execute
```