# The Story of Open Source, Linux, and the Languages That Built Our World

*Concise lecture notes for CS students — told as a story, because that is what it is.*

---

## Prologue: What Does "Open" Actually Mean?

Before we trace the arc of open source software, we need to be precise about what the word means. "Open source" does not mean "free of cost." It means the **source code** — the actual text files written in C, Python, or any other language — can be run, examined, modified, and redistributed by anyone. These rights are granted through **licenses**, and the entire philosophical and legal landscape of OSS turns on which license you choose.

There are two schools of licensing. **Permissive licenses** (MIT, Apache 2.0) allow derivative works to be closed-source; they maximize adoption. MIT is the most minimal ("do whatever you want, just keep the copyright notice"), while Apache 2.0 adds explicit patent grants, which is why research labs and infrastructure projects (think Google's TensorFlow) prefer it. **Copyleft licenses** (GPL family) are restrictive by design: any derivative work must also be open-sourced under the same terms. The GPL is a legal instrument that enforces openness virally — if you use GPL code in your product, your product must also be GPL. This distinction — permissive vs. copyleft — is the single most consequential axis in OSS strategy.

Why does source code matter? Because sharing only the compiled executable (the binary) lets you *run* the program but not *study, modify, or redistribute* it. That is the definition of proprietary software. Open source requires that the source code itself is available.

A quick clarification on the landscape: OSS does not require an OSS operating system. Apache Web Server and VLC run perfectly well on Windows. Conversely, closed-source software runs on Linux every day. And closed-source software can be free of cost (Adobe Acrobat Reader), just as open-source software can be sold commercially (Red Hat Enterprise Linux). The axes of "open vs. closed" and "paid vs. free" are orthogonal.

---

## Act I: Before Software Was a Product (1950s–1970s)

### The Bundled Era

In the 1950s, software did not exist as a separate commercial entity. It was tightly coupled to hardware — sold as the thing you needed to operate the machine, bundled at no extra charge. Binaries compiled for one machine were useless on another. There was no portability, no reuse, no ecosystem.

### ARPANET and the Dawn of Collaboration

In the 1960s, the U.S. Department of Defense funded ARPANET (1968), a distributed network connecting research computers. For the first time, developers at MIT, UC Berkeley, AT&T Bell Labs, and Xerox PARC could share code and collaborate remotely. This infrastructure was the precondition for everything that followed.

### UNIX: The Operating System That Changed Everything

At Bell Labs in 1969, **Ken Thompson** wrote UNIX in assembly language. The philosophy was radical for its time: small programs that each do one thing well, composed together via pipes and files. "Everything is a file."

But assembly is ISA-specific — the OS was welded to one hardware architecture. This is where **Dennis Ritchie** enters. He developed the **C programming language** (1972–73) and rewrote UNIX in C, making it **portable**: write once, compile on different hardware. Before this, the standard practice was to rewrite the entire operating system from scratch for every new machine. C solved the portability problem, and UNIX solved the modularity problem. Together, they created the template for modern systems software.

The key insight: C was not designed to be elegant or safe. It was designed to be *close enough to the metal* to write an operating system, while being *abstract enough* to compile across architectures. That trade-off — power and portability at the cost of safety — defined systems programming for the next fifty years.

---

## Act II: The Academic Golden Age (1970s–1983)

### BSD: Berkeley Takes UNIX and Runs

AT&T, constrained by antitrust regulation, could not sell software commercially. So they gave UNIX to **UC Berkeley**, where researchers improved it into **BSD** (Berkeley Software Distribution). BSD added the TCP/IP networking stack, virtual memory, and an improved file system — contributions so fundamental that Apple later used BSD as the foundation for macOS. The lineage runs: UNIX → BSD → Darwin → macOS. Every time you open a Mac terminal, you are touching code whose ancestry traces back to 1970s Berkeley.

### TeX and Emacs: Two Seeds

At Stanford, **Donald Knuth** created **TeX** (1978), a digital typesetting system of extraordinary precision — still the standard for academic publishing nearly fifty years later.

At MIT, **Richard Stallman** created **Emacs**, a programmable text editor that became one of the first major collaborative software projects. Dozens of contributors wrote extensions, shared configurations, and built on each other's work. This experience planted a seed in Stallman's mind: *large-scale open collaboration on software is not just possible — it produces better results.*

---

## Act III: The Free Software Movement (1983–1991)

### The Trigger

In the early 1980s, companies realized software could be sold independently of hardware — and began closing their source code. AT&T reversed its open policy on UNIX, restricted access, and sued BSD. Stallman, who had watched the MIT AI Lab's culture of open sharing erode as companies hired away researchers and locked down code, decided to fight back.

### GNU and the Four Freedoms

In **1983**, Stallman launched the **GNU Project** (a recursive acronym: "GNU's Not UNIX") — an audacious plan to build a complete, free, UNIX-compatible operating system from scratch. The name was deliberately chosen to signal: *this is UNIX-like, but it is not UNIX — it is free.* In **1985**, he published the **GNU Manifesto** and founded the **Free Software Foundation (FSF)**, which codified four freedoms: the freedom to run, study, modify, and redistribute software.

By the late 1980s, GNU had produced an impressive suite of tools: **GCC** (a compiler collection for C, C++, Fortran), **GDB** (a debugger), **Bash** (the shell), **Coreutils** (the everyday UNIX command-line utilities), and **GIMP** (image editing). Every time you type `ls`, `cat`, or `grep` in a Linux terminal, you are using GNU software.

### GPL: The Legal Innovation

In **1989**, Stallman created the **GNU General Public License (GPL)** — the first copyleft license. Its mechanism is elegant and subversive: you may use, modify, and redistribute the code, but any derivative work must also be released under the GPL. This means openness propagates virally. The GPL was not just a license; it was a legal weapon designed to prevent the re-enclosure of free software.

### The Missing Piece

GNU had compilers, debuggers, shells, and utilities. It had everything except the most critical component: a **kernel** — the privileged core of the operating system that mediates between user programs and hardware. The GNU project's own kernel (Hurd) was perpetually delayed due to its ambitious microkernel design. The entire free software stack was waiting for someone to write a working kernel.

---

## Act IV: Linux and the Explosion (1991–2000)

### Linus Torvalds and the Kernel

In **1991**, a 21-year-old computer science student at the University of Helsinki named **Linus Torvalds** wanted to learn operating system design. He was dissatisfied with Minix, a teaching OS by Andrew Tanenbaum that had licensing restrictions preventing modification. So he wrote his own kernel — the **Linux kernel** — and released it under the **GPL**.

The timing was perfect. GNU had all the userspace tools but no kernel. Linux was a kernel with no userspace tools. Combined, they formed a complete, free operating system: **GNU/Linux**. This combination, bundled with different desktop environments, package managers, and configuration philosophies, produced what we call **Linux distributions**.

### Python Arrives (1991)

In the same year, independently, **Guido van Rossum** released **Python** — designed from the ground up for simplicity, readability, and rapid prototyping. Python's philosophy ("there should be one obvious way to do it") was the polar opposite of C's philosophy ("give the programmer maximum power and trust them not to shoot themselves in the foot"). Python did not aim to replace C for systems programming. It aimed to make programming accessible to a vastly larger population of people — scientists, analysts, students, and eventually machine learning researchers who would reshape the industry two decades later.

### The Distribution Family Tree

Linux distributions proliferated along distinct philosophical lines:

**Debian** (1993), created by Ian Murdock (the name combines his then-girlfriend Debra's name with his own), was community-driven and committed to fully free software. It became the most stable and widely-forked distribution, fathering **Ubuntu** (2004, by Canonical, aimed at making Linux beginner-friendly), **Kali Linux** (security testing), and **Raspberry Pi OS**.

**Red Hat** (1994) proved that open source could be a successful business. Red Hat Enterprise Linux (RHEL) provided stability, security, and paid support for enterprise customers — generating billions in revenue before being acquired by IBM. Its descendants include **Fedora** (Linus Torvalds' personal distribution of choice) and **CentOS**.

**Arch Linux** (2002) took the opposite approach: minimalism, simplicity, and user control, with its rolling-release model and the `pacman` package manager.

```
Linux Kernel
         |
    ┌────────────┬──────────────┬──────────────┐
  Debian       Red Hat       Slackware      (others)
  (1993)       (1993)        (1993)
    |             |              |
 ┌──┴───┐    ┌───┴───┐      Arch (2002)
Ubuntu  Kali  Fedora CentOS
(2004) (2013) (2003) (2004)
```

### 1994–1998: The Infrastructure Wave

Several technologies emerged in this period that cemented Linux and OSS as the backbone of the internet:

**Apache HTTP Server** (1995) — nicknamed "a patchy server" for its origin as a collection of patches — became the dominant web server. The growth curves of Apache, Linux adoption, and internet usage track each other almost perfectly. Apache was the "killer app" that gave administrators a concrete reason to deploy Linux on servers.

**KDE** (1996) and **GNOME** (1997) made Linux usable for non-technical users by providing graphical desktop environments. GNOME's creation is itself an instructive story: KDE was open-source, but it depended on the **Qt toolkit**, which had a proprietary license. GNOME was built to ensure the entire stack — from kernel to desktop — was free. This is the kind of principled, layer-by-layer freedom that Stallman's movement demanded.

**Netscape** (1998) released its browser source code, which became **Mozilla**, and eventually **Firefox**. This was a landmark: the first major corporate open-source release, driven not by idealism but by competitive desperation. Netscape feared that if Microsoft's Internet Explorer monopolized the browser, it would lock developers into Microsoft's server ecosystem. Open-sourcing the browser was a strategic countermove.

### 1998: The Term "Open Source" Is Coined

In **1998**, pragmatists led by **Eric Raymond** (author of "The Cathedral and the Bazaar") coined the term "open source" to rebrand the movement for business audiences. Stallman's "free software" carried connotations of anti-commercial ideology that made corporations nervous. "Open source" emphasized the practical benefits: faster development, more contributors, better debugging. Raymond's famous dictum captured the core promise: **"Given enough eyeballs, all bugs are shallow."**

The movement now had two wings: the **idealistic wing** (Stallman, FSF — software should be free as a matter of ethics, like knowledge or medicine) and the **pragmatic wing** (Raymond, OSI — open source produces better software faster, and that is reason enough).

---

## Act V: Corporate Open Source and Global Dominance (2000–Present)

### The Big Releases

**Ubuntu** (2004) — Mark Shuttleworth's mission to make Linux accessible to ordinary users. "Linux for human beings."

**Git** (2005) — Linus Torvalds, frustrated with the existing version control tools for Linux kernel development, wrote Git in a matter of weeks. Git's distributed model (every developer has a full copy of the repository history) was the architectural insight that enabled GitHub, GitLab, and the modern pull-request workflow that defines collaborative software development.

**Android** (2007) — Google built the world's most widely used operating system on top of the Linux kernel. Every Android phone runs Linux. This is arguably the single largest deployment of open-source software in human history.

**Kubernetes** (2014) — Google open-sourced its container orchestration system, enabling the cloud-native infrastructure that runs most modern web services.

### The Revenue Model Paradox

How do companies make money from software that is free to use? The dominant model, pioneered by Red Hat and refined by companies like Canonical (Ubuntu) and Automattic (WordPress), is **support and services**: the software is free, but enterprises pay for reliability guarantees, security patches, consulting, and managed hosting. This unbundled the traditional closed-source model, where the vendor monopolized both the software and the support. OSS made support an open, competitive market.

### The Cautionary Tales

Not every OSS story is a triumph. **MySQL** was open-source, then acquired by Oracle and increasingly steered toward closed-source commercial licensing — prompting the community to fork it into **MariaDB**. **Mozilla Firefox** traveled the opposite direction: from Netscape's closed-source browser to a fully open-source project. These trajectories remind us that "open" and "closed" are not permanent states; they are strategic decisions that shift with corporate ownership, market pressure, and community dynamics.

---

## Interlude: The Programming Language Lineage

Each major language in this story was created to solve a specific pain point. Understanding *why* each language was invented is more important than memorizing *when*.

### C (1972) — Dennis Ritchie, Bell Labs

**Pain point:** Operating systems had to be written in assembly, which was architecture-specific and non-portable. **Key idea:** A language close enough to hardware to write a kernel, abstract enough to compile across architectures. **Trade-off:** Power and portability at the cost of manual memory management and safety. **Legacy:** Systems programming, embedded systems, and the foundation of nearly every language that followed.

### C++ (1983) — Bjarne Stroustrup, Bell Labs

**Pain point:** As software grew more complex (games, browsers, simulations), C's procedural paradigm made it difficult to manage large codebases with many interacting components. **Key idea:** Add object-oriented programming (classes, inheritance, polymorphism) to C, enabling better abstraction and code organization for large-scale projects while retaining C's performance. **Trade-off:** Enormous language complexity. **Legacy:** Game engines, browsers (Chrome, Firefox), high-frequency trading, performance-critical applications.

### Python (1991) — Guido van Rossum

**Pain point:** Programming was inaccessible to non-specialists. C and C++ demanded deep knowledge of memory management and compilation. **Key idea:** Readability as a first-class design principle. An interpreted language (no compilation step) with clean syntax, dynamic typing, and automatic memory management. **Trade-off:** Runtime performance, which turned out to matter less than anyone expected for most use cases. **Legacy:** Data science, machine learning, scripting, web development, and the default "first language" for a generation of programmers.

### Java (1995) — James Gosling, Sun Microsystems

**Pain point:** Embedded devices needed portable software, and C/C++ compiled to native machine code that was not portable across hardware. **Key idea:** Compile to intermediate **bytecode** that runs on a **Java Virtual Machine (JVM)** hosted on any hardware. "Write once, run anywhere." Also introduced **automatic garbage collection**, eliminating the need for manual `free()` calls. **Trade-off:** JVM overhead and verbose syntax. **Legacy:** Enterprise backends, Android (initially), and the entire JVM ecosystem.

### Scala (2004) — Martin Odersky, EPFL Lausanne

**Pain point:** Java was verbose and lacked functional programming features that enabled safer concurrent and distributed computation. **Key idea:** A language that fuses object-oriented and functional programming on the JVM, with type inference to reduce boilerplate. **Legacy:** Apache Spark, data engineering at scale, and a proof that the JVM could host languages with very different philosophies than Java.

---

## Act VI: Linux — From Boot to Shell

Since most servers in the world run Linux distributions, every computer scientist needs to be able to SSH into a Linux machine and work effectively. Here is the essential mental model of what happens when a Linux machine starts, and how you interact with it.

### The Boot Sequence

When you power on a Linux machine, the bootloader (**GRUB**) loads the Linux kernel — a compiled C executable — into RAM. The kernel initializes hardware, mounts the root filesystem, and creates the **init process** (`systemd` on modern systems), which is the ancestor of every user process on the system. It has PID 1, and every other process is its descendant.

### The Kernel's Role

The kernel enforces a strict boundary between **user space** (where your programs run, in restricted mode) and **kernel space** (privileged mode, with direct hardware access). User programs cannot touch hardware directly; they must request services from the kernel via **system calls**. The **GNU C Library (glibc)** provides wrapper functions that make these system calls ergonomic — when you call `open()`, `read()`, or `write()` in a C program, glibc translates that into the appropriate system call.

Every familiar shell command (`ls`, `cat`, `touch`, `grep`) is a GNU utility that, under the hood, issues system calls. The kernel checks permissions, uses the appropriate device drivers, and executes the requested operation.

### POSIX Compliance

UNIX's influence led to a standardization effort called **POSIX** (Portable Operating System Interface), which defines a common API for UNIX-like operating systems. macOS, Linux, and Android are POSIX-compliant. Windows is not, which is why shell scripts and UNIX tools behave differently (or fail entirely) on Windows without a compatibility layer like WSL.

### The Filesystem Hierarchy

Linux organizes everything into a single directory tree rooted at `/`. Key directories and their purposes:

`/boot` holds the kernel image and bootloader configuration. `/bin` and `/sbin` contain essential user and system binaries (the commands you need to boot and repair the system). `/etc` stores system-wide configuration files. `/home` contains user home directories. `/dev` exposes hardware devices as files (consistent with the UNIX "everything is a file" philosophy). `/proc` and `/sys` are virtual filesystems that expose kernel and process information. `/var` holds variable data like logs and mail queues. `/tmp` is for temporary files. `/usr` contains user-installed programs and libraries (most of what you install via a package manager lands here).

### Shell Configuration

When you open a terminal, Bash reads configuration files — most importantly `~/.bashrc` — before presenting you with a prompt. This is where you set environment variables (like `$PATH`, which tells the shell where to find executables), define aliases, and configure your working environment. If you install a new tool and the shell cannot find it, the first thing to check is whether its binary directory is in your `$PATH`.

---

## Epilogue: The Stakes

The story of open source is not just a history of software. It is a story about whether knowledge should be enclosed or shared, whether infrastructure should be controlled by a few or auditable by all, and whether the tools we depend on — from operating systems to machine learning frameworks — should be transparent or opaque.

Every time you `git clone` a repository, compile code with GCC, run a Python script, or deploy a container on Kubernetes, you are standing on layers of infrastructure that exist because people chose to share their work. Understanding this history is not optional for a computer scientist. It is the context that makes the present legible.

---

*Topics flagged for deeper study: process monitoring and control (`ps`, `htop`, signals like `SIGKILL`, `SIGTERM`), system daemons and background processes, cron jobs and scheduling, the ext4 filesystem in detail, and the full lifecycle of a process (creation, scheduling, context switching, termination).*