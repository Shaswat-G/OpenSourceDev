1. What is OSS and licensing?
   Open Source is a way to define the "freeness" in the word "free" - It is not free of cost, but the source code of the S/W can be accessed, examined, modified and redistributed. These rights of use are provided via "licensing," which has two major schools --> Permissive or Restrictive. Permissive is MIT (maximal adoption), Apache (Research labs for infra projects) vs Restrictive (copy-left GPL licences). This is based on whehter the changes made (and derivative products) are also open soruce or can be closed source.

   Source Code is simply the collection of code files as text in a compiled language (C/C++ that is compiled into object code or binaries or executable code) or interpreted language (Ptyhon, no compilation required, the interpreter can run it line by line, same for shell script and javascript in browser) with comments. Sharing the executable alone is not OSS (only run), since user needs the source code to study, modify and redistribute.

   Pragmatism Angle: OSS dev leads to be faster and better in dev due to more contributors, reviewers, easier debugging.
   Idealisitc Angle: S/W should be "open" like knowledge for ideological and ethical reasons beyond technical reasons. (medical devices)

   Transparency (you are not hiding anything), Trust, white box.

   Eg: LibreOffice, OpenOffice, VLC media player, GIMP (for picture processing), Audacity, etc.

   OSS do not necessarily need an OSS OS  like Linux, Eg: Apache Webserver or VLC can run on windows. Vice versa, closed source s/w can be run on oss os.
   Closed source S/w can be free to (like Adobe Acrobat reader)

   Freeware vs TrialWare (time limited and or feature limited)

2. What is Proprietary Software?
   S/W that only the "S/W owners" have full legal access to. Trusted Partners maybe allowed to inspect and examine the code under an NDA. Owners may not be the authors of that code. Historically, this was the only model used for commercial projects (windows). Therefore, this is closed-source. The nature of restrictions made by licensing here is very different. Here, licenses typically restrict redistribution, reconstruction and usage in other products, while protecting the author from the misuse/damages produced by the product.

   MS windows, MS Office provide executables whose source code is closed (proprietary)

   Note about revenue model: OSS sfotware, companies provide support around this ecosystem (eg: RedHat and Wordpress hosting with support from WP engine?)

   MySQL (OSS to Closed source)
   Mozilla Firefox (closed source to OSS)

   We need to have proper case studies to see understand the landscape evolution!
   
3. History to modern times:
   **Pre-Commercial Compute Era leading to Internet and UNIX, 1950-1970**
   Rapid development in CS, both H/W and S/W.
   1950s - S/W tightly coupled to H/W, S/W sold as something to operate the H/W, bundled in free of cost. S/W for one H/W does not run on any other (binaries were not portable)
   1960s - Arpanet from US Defnese (distributed network connecting research computers in 1968) evolved into the internet and allowed developers to share and contribute to S/W to collaborate in build, MIT and Berkley (academia) + AT&T Bell Labs and Zerors (Industry Research). UNIX (everything is a file)
   Ken Thompson who wrote UNIX in assembly in 1969? Dennis Ritchie re-wrote it in C to make it portable (1973)
   Key problem solved was (before UNIX, OS were huge, monolithic and H/W specific so not portable). Portability and Modularity added.
   Philosophy was: Have small programs that do one thing very well. Combine programs together (composition)

   **Academic UNIX 1970-1980**
   AT&T was not allowed to sell software commercially, so they gave it for free to UC Berkley where researchers improved UNIX into BSD (Berkley Software Distribution). They added TCP/IP networking stack, virtual memory and improved file system --> This was taken by Apple MacOS foundation to develop MacOS on top of open source BSD?
   At Stanford, Donald Knuth creates Tex (advanced digital typesetting.)
   At MIT, Richard Stallman creates a programmable text editor called Emacs --> first major collaboration based S/W project that gave Stallman the motivation for large scale open collaboration.

   **Free Sofware Movement 1983-1991**
   Companies realized that they could sell software, and began closed sourcing (Eg: AT&T closed Unix source access and actually sued BSD) --> triggered a social movement.
   In 1983, Richard Stallman starts the GNU project (recursive acronym, GNU is not Unix) to create a completely free UNIX-like OS to replace all proprietary UNIX tools (gatekept by AT&T?). He had soured experiences in MIT's AI lab about closed S/W.

   GNU introduced:
    1. GCC (compiler for languages)
    2. GDB (debugger to step through and find bugs)
    3. Bash (shell scripting, commands)
    4. Coreutils (Unix utilities)

    But, it was missing a single core piece that separateed user programs (restricted mode) from the kernel (priveleged mode) that interacts with the H/W, which was later developed by Linus Torvalds in 1991 under GNU's GPL (general public lincense, essentially all changes and derivatives have to be open sourced). His inspiration came from SunOS?
    In 1985, he founded the Free Software Foundation (FSF) advocating for 4 freedoms (run, study, modify, redistribute)  while supporting development for GNU.

    I think Stallman must have chosen the name GNU to differentiate agains unix (closed source by AT&T) with a recursive name to appeal to devs?

    In 1989, he created a legal instrument called GNU GPL (copyleft) which says any open source work must remain open sourced

    **Birth of Linux 1991-1998**
    At Univ of Helsinki, 21 year old Linus Torvalds wanted an OS kernel to learn OS system design to avoid using Minix a teaching OS by Anrew Tanenbaum, which also had licnesing restrictions. He released Linux Kernel in 1991, alogn with the GPL licensing and collaboration with GNU it became a fully functional OS (GNU + Linux Kernel). This is bundled in a "distribution" with differnet control philosophies, GUIs, etc to make Linux Distributions.

    In 1991, Python was created indepenedlty to be simple, readable and highly productive.

    Debian was created by Ian Murdock in 1993 with 2 tenets (community driven + fully free S/W). It served as a direct ancestor to Ubuntu and Kali Linux.

    In 1994, RedHat proved that **OSS could be a succesful business** (RHEL) with support --> BigTech like Microsoft felth threatened by Linux (Halloween Latters) etc were agains OSS, but later embraced it to get the community support (by positioning themselves as allies), debugging, quality software while insidiously funding and directing the community to buuild features that help them. Closed source had monopolized support service, but, OSS made support open and competitive.

    Eric Raymond met MS's VP of commerical products.

    IN 1998, the term "open source" was coined. "Given enough eyeballs, all bugs are shallow.". This is where the promise of OSS is, the massive, independent peer reviews! It really works.

    Many major technologies emerged in this decade.
    5. Apache Web server ("a patchy server") - became the dominant web server in 1995. The growth of the internet, ISPs, Linux adoption follow the same curve. This Apache server added the "functionality" value add that compelled people to shift.
    6. KDE (K Desktop Env) - make Linus user friendly with a GUI.
    7. GNOME (1997) - fully open alternative to KDE.
    8. Netscape releases its browser source code --> Mozilla --> FIrefox. (large corporate open source releases) to compete with MS's internet explorer. They were worried that if MS controlled browsers and locked in this thing, it could lead to loss in its server market.

    Debain (Ian murodckl + wife Debra) was the most popular (Linux Kernel + GNU distro) that was stable and fathered many offspring distros including RaspberryPi OS. It created Ubuntu, which is by far the most popular debian based distro (managed by a british company calleed Canonical) to make it beginner friendly

    Another big family was proliferated by RedHat (RHEL) for security and reliability for enterprise users. Billions of dollars of revenye and acquired by IBM. Children: RHEL, CentOS, Fedora (the distro of choice by Linus Torvalds).

    In the early 2000s, Arch Linux came out for simplicity and minimalism (pac-man package manager).

    **Corporate OpenSource 2000-2005**
    9.  Ubuntu (2004) - Linux that the common man can use.
    10. Git (2005) - Linus Torvalds, again for Linux developers for version control system.
    11. Android (2007) - Devloped by Google based on the Linus Kernel. (the most widely used OS in the world)
    12. SImilary for Kubernetes (again by Google) for container orchestration.
    
    Since most of the servers run Linux distros, you have to be able to ssh into one and debug.



4. OSS Licenses:
   


5. Importance of sharing and collaborating from stakeholder classes (univ, businesses, devs, students, individual users)


6. Linux basics to get started:
   Unix led to a standardization called POSIX - Portable Operating System interface. MacOS, Linux and android are POSIX compliant while windows is not.

   Bootloader (GRUB) loads the C-executable linux kernel onto RAM.
   The Kernel creates the init process (which is the first user process with many names including systemd, the ancestor or root of all procesess with pid=1)
   Provides the interface for user processes to interact with H/W.

   GLIBC provides interfaces and subroutines as wrappers to systemcalls that can be made in the program and do many things.
   All the bash utilites that we use (like touch, cat, etc) are actually GNU tools that under the hood make a system call that checks for priveleges, uses the right device drivers to get the job done.




Learn about fourth extended file system (the typical folder structure and what do you expect to find in each one like boot, dev, etc, bin and sbin)
I want to know the basiscs of running commands (processes, threads, compilers, signals, stdout and stdin, error codes etc)

Bash rc is a file that runs before very terminal session, that sets your env variables like path to point towards the binaries of the stuff you just installed.


I want to learn more about process monitoring and control (ps, htop, etc and signals like sigkill etc)
I also want to learn more about system daemons, background processes and cron jobs. 
