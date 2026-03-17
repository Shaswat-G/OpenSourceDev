Add common recipes for each file systems and help the reader understand the scope of what and why for each directory and with examples explain the use-case recipes. Expand on the this too.


Linux is UNIX-like in the sense that it is POSIX compliant, which means its file system and directory structure has a well-defined function.

Windows and Linus evolved differently.

MS DOS (Disk Operating System) - this was command line only, but you could stil run programs and games. Windows was a desktop environment (GUI) that ran on top of MS DOS with the win command. Disks "A" and "B" were reserved names for removable drives sincle floppy disks could be removed. As Hard drives came along, the letter "C" became the letter to represent the internal disk, and the next consecutive letters were used for more drives ("D", "E" etc).

Windows evolved to be less and less dependent on DOS with an independent kernel which allowed windows to boot without DOS at all. With windows 95, microsoft installs all its programs in c:\Program Files, with a separate folder for each program cotainined executables with dlls? (what are dlls?)

Linux follows unix traditions of using front slashes with case sensitive file names (Eg: file File fiLe, etc are different).

### /Home
Every user has a different home directory where all their files are stored. Documents, Downloads, etc. 
Settings are stored in hidden files here .cache, .config, .local (individual app configs). User specific configuration files for applications are stored in the user's home directory in a file that starts with the '.' character (a "dot file").

### /bin
Stands for binaries. Essential command binaries (like cat, ls, grep, zip) are housed here.

### /sbin
Stands for system binaries that are used by the system administrator that the normal user does not have access to. 

### /boot
Static files for Firmware, UEFI, Bootloaders (GRUB) liver here. We never need to open it or modify it. Contains data and code used before kernel begins executing user-mode programs.

### /dev
Stands for devices. There is a file for each hardware entity. Eg: Disk partitions are sda0, sda1, sda2, etc.
Webcam and keyboard here too. APplications and drivers will access this, not user.

### /etc
Host specific system configuration. Editable configurable files (originally meant to the et-cetera). System wide things. Like certificates, crons, package manager repo lists + settings.
Any system-wide setting will be here.

### /lib, /lib32, /lib64
Essential shared libraries and kernel modules which store the code required by binaries and system binaries to run.

### /media and /mnt
Here is where we can find other mounted drives (Media is used as a mount point for removable drives like network drives, pen drive, another disk drive, etc). The OS manages the media directory. You as the user manually mount temporary file systems to /mnt.

### /opt
Optional folder. Manually installed software can be found here. This where we can store our own applications for linux.

### /proc
Pseudo files that expose processes as files that contain info about resources. get the PID of the process and you can get all the "status" of that process.
> cat /proc/cpuinfo
>

### /root
??

### /run
This is a tempfs file system (it runs on RAM?). => Not persistent across boots?

### /snap
Used by ubuntu for snap applications that are self contained. ??

### /srv
Running a server here, the files that you want to make accessible via the server will be stored here.

### /sys
Its a way to interact with the kernel. Not physically written to the disk, not persistent.

### /tmp
For logs. A temporary copy of unsaved work is saved here.

### /usr
User application space. This is where user applications will be installed unlike bin and sbin used by the system. These are non-essential for system operation. Further has /usr/bin and sbin, also /usr/local/bin and sbin, with libraries required store in usr/lib or usr/local/lib.

### /var
Expected to grow in size. /var/log, /var/tmp, /var/crash






The Linux Foundation maintains [FHS](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html) (File Hierarchy Standard) which delineates structure and composition. (membership criteria?)


This standard allows both user and software to predict the location of installed files and directories. It describes guiding principles for each area of the filesystem, specify the minimum directories and files required and enumerate exceptions to principles.

2 key features of files drive principles:
1. Shareable vs Non-Shareable : Files in the user home direcotries are shearable whereas device lock files (??) are not.  
2. Static vs Variable : Static files include binaries, libraries, documentation fiels that do not change withour sysad intervention. Useful to differentiate since variable files will have a more frequent backup schedule, and static files can be read only. /var waas created and all variable files from /usr and /etc were shifted here.

static + shareable : /usr and /opt
static + unshareable : /etc and /boot
variable (shareable or unshareable) : /var

