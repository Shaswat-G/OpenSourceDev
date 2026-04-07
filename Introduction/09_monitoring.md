There are many tools to monitor system performance like top, cpu perfomrance, memory usage, network performance, etc.


top: It is a command-line utility that provides a dynamic real-time view of a running system. It displays system summary information and a list of tasks currently being managed by the Linux kernel. You can use it to monitor CPU usage, memory usage, and other system resources.
htop: It is an interactive process viewer for Unix systems. It is a more user-friendly alternative to the top command, providing a more visually appealing interface and additional features such as mouse support and the ability to scroll through the list of processes.
free: It is a command-line utility that displays the amount of free and used memory in the system. It provides information about total memory, used memory, free memory, shared memory, buffer/cache memory, and available memory.
vmstat: It is a command-line utility that provides information about system performance, including CPU usage, memory usage, and disk I/O. It can be used to monitor system performance over time and identify performance bottlenecks.
iostat: It is a command-line utility that provides information about CPU usage and input/output statistics for devices and partitions. It can be used to monitor disk performance and identify potential issues with storage devices.
netstat: It is a command-line utility that provides information about network connections, routing tables, and network interface statistics. It can be used to monitor network performance and identify potential issues with network connections.


Kernel Modules:
You can use kernel modules (add or remove them at runtime).

udev?

## User and Group Management:
In Linux, user and group management is essential for controlling access to resources and ensuring system security. Here are some common commands and tools for managing users and groups:
1. useradd: This command is used to create a new user account. It allows you to specify various options such as the user's home directory, shell, and group membership.
2. usermod: This command is used to modify an existing user account. You can use it to change the user's home directory, shell, group membership, and other attributes.
3. userdel: This command is used to delete a user account from the system. It can also be used to remove the user's home directory and mail spool.
4. groupadd: This command is used to create a new group. You can specify the group name and GID (Group ID) when creating a new group.
5. groupmod: This command is used to modify an existing group. You can use it to change the group name and GID.
6. groupdel: This command is used to delete a group from the system. It can also be used to remove the group from any user accounts that are members of the group.
7. passwd: This command is used to change a user's password. It can be used by both users and administrators to update their passwords.
8. id: This command is used to display the user ID (UID), group ID (GID), and group memberships of a user. It can be used to verify the user's identity and group affiliations.
9. whoami: This command is used to display the current user's username. It can be useful for verifying the user's identity and ensuring that you are logged in as the correct user.
10. groups: This command is used to display the groups that a user belongs to. It can be used to verify the user's group memberships and ensure that they have the appropriate permissions to access resources.
11. su: This command is used to switch to another user account. It allows you to run commands with the privileges of another user, which can be useful for administrative tasks or testing user permissions.
12. sudo: This command is used to execute a command with superuser privileges. It allows authorized users to perform administrative tasks without needing to switch to the root user account. It is a safer alternative to using the su command for administrative tasks, as it provides better control over which users can execute specific commands with elevated privileges.
13. chown: This command is used to change the ownership of a file or directory. It allows you to specify the new owner and group for the file or directory, which can be useful for managing permissions and access control.
14. chmod: This command is used to change the permissions of a file or directory. It allows you to specify the read, write, and execute permissions for the owner, group, and others. This is essential for controlling access to files and directories and ensuring system security.
15. chgrp: This command is used to change the group ownership of a file or directory. It allows you to specify the new group for the file or directory, which can be useful for managing permissions and access control based on group memberships.


The first non-root user has UID 1000, and the first group has GID 1000. The root user has UID 0 and GID 0. The /etc/passwd file contains user account information, while the /etc/group file contains group information. The /etc/shadow file contains encrypted password information for user accounts. The /etc/sudoers file contains configuration for sudo access control.

Usually the UID and GID of a user are the same, but they can be different if needed. The useradd command can be used to create a new user account, and it will automatically create a group with the same name as the user and assign the same UID and GID to both the user and the group. However, you can use options with the useradd command to specify different UIDs and GIDs if necessary.

Groups are a set of users with a common shared goal. They are used to manage permissions and access control for resources on the system. By assigning users to groups, you can control which users have access to specific files, directories, and other resources based on their group memberships. This allows for more efficient management of permissions and helps to ensure that users only have access to the resources they need.

## Package Management:
Package management is a crucial aspect of maintaining a Linux system, as it allows you to easily install, update, and remove software packages. Different Linux distributions use different package management systems, but here are some common commands and tools for package management:
1. apt: This is the package management tool used by Debian-based distributions such as Ubuntu. It provides a command-line interface for managing packages, including installing, updating, and removing software. Some common apt commands include:
   - apt update: This command is used to update the package index, which allows the system to know about the latest available packages and their versions.
   - apt install <package_name>: This command is used to install a specific package. You can specify the package name to install the desired software.
   - apt upgrade: This command is used to upgrade all installed packages to their latest versions. It will update the packages based on the information in the package index.
   - apt remove <package_name>: This command is used to remove a specific package from the system. It will uninstall the package and remove its associated files.
   - apt autoremove: This command is used to remove packages that were automatically installed to satisfy dependencies for other packages and are no longer needed. It helps to clean up the system by removing unnecessary packages.
2. yum: This is the package management tool used by Red Hat-based distributions such as CentOS and Fedora. It provides a command-line interface for managing packages, including installing, updating, and removing software. Some common yum commands include:
   - yum update: This command is used to update all installed packages to their latest versions. It will check for updates and install them if available.
   - yum install <package_name>: This command is used to install a specific package. You can specify the package name to install the desired software.
   - yum remove <package_name>: This command is used to remove a specific package from the system. It will uninstall the package and remove its associated files.
   - yum autoremove: This command is used to remove packages that were automatically installed to satisfy dependencies for other packages and are no longer needed. It helps to clean up the system by removing unnecessary packages.
3. dnf: This is the package management tool used by newer versions of Red Hat-based distributions such as Fedora. It is a modernized version of yum and provides similar functionality for managing packages. Some common dnf commands include:
   - dnf update: This command is used to update all installed packages to their latest versions. It will check for updates and install them if available.       
    - dnf install <package_name>: This command is used to install a specific package. You can specify the package name to install the desired software.
    - dnf remove <package_name>: This command is used to remove a specific package from the system. It will uninstall the package and remove its associated files.
    - dnf autoremove: This command is used to remove packages that were automatically installed to satisfy dependencies for other packages and are no longer needed. It helps to clean up the system by removing unnecessary packages.
4. pacman: This is the package management tool used by Arch Linux and its derivatives. It provides a command-line interface for managing packages, including installing, updating, and removing software. Some common pacman commands include:
   - pacman -Syu: This command is used to update the package database and upgrade all installed packages to their latest versions. It will check for updates and install them if available.
   - pacman -S <package_name>: This command is used to install a specific package. You can specify the package name to install the desired software.
   - pacman -R <package_name>: This command is used to remove a specific package from the system. It will uninstall the package and remove its associated files.
   - pacman -Rs <package_name>: This command is used to remove a specific package along with its dependencies that are no longer needed. It helps to clean up the system by removing unnecessary packages and their dependencies.
5. zypper: This is the package management tool used by openSUSE and SUSE Linux Enterprise. It provides a command-line interface for managing packages, including installing, updating, and removing software. Some common zypper commands include:
   - zypper refresh: This command is used to refresh the package repository information, allowing the system to know about the latest available packages and their versions.
   - zypper install <package_name>: This command is used to install a specific package. You can specify the package name to install the desired software.
   - zypper update: This command is used to update all installed packages to their latest versions. It will check for updates and install them if available.
   - zypper remove <package_name>: This command is used to remove a specific package from the system. It will uninstall the package and remove its associated files.
   - zypper clean: This command is used to clean up the package cache and remove any unnecessary files. It helps to free up disk space by removing cached packages and metadata that are no longer needed.

6. rpm: rpm is a low-level command designed for individual packages listed on the command line or for groups of packages listed on the command line. It does not by itself solve package dependency issues. It is used to install, update, remove, and query individual packages. Some common rpm commands include:
   - rpm -i <package_file.rpm>: This command is used to install a package from an RPM file. You can specify the path to the RPM file to install the desired software.
   - rpm -U <package_file.rpm>: This command is used to upgrade a package from an RPM file. It will replace the existing package with the new version specified in the RPM file.
   - rpm -e <package_name>: This command is used to remove a package from the system. You can specify the package name to uninstall the desired software.
   - rpm -q <package_name>: This command is used to query a package to check if it is installed on the system. It will display information about the package if it is installed, or indicate that it is not installed if it is not found.
7. snap: This is a package management system developed by Canonical for installing and managing software packages in a containerized format called snaps. It provides a command-line interface for managing snap packages, including installing, updating, and removing software. Some common snap commands include:
   - snap install <package_name>: This command is used to install a snap package. You can specify the package name to install the desired software.
   - snap refresh <package_name>: This command is used to update a snap package to its latest version. It will check for updates and install them if available.
   - snap remove <package_name>: This command is used to remove a snap package from the system. It
    will uninstall the package and remove its associated files.
   - snap list: This command is used to list all installed snap packages on the system. It will display the package name, version, and other relevant information for each installed snap package.



## Logging:
The logging pipeline is a sequence of transformations:

Source (kernel, daemon, app) → generates message
Transmission (syslog protocol) → /dev/log socket
Routing (rsyslogd rules) → matches facility + severity
Action (output) → file, pipe, network, terminal, discard
Lifecycle (logrotate) → compress, archive, delete
Access (tail, grep, journalctl) → human-readable retrieval

Key principles:

Facilities = WHO is logging (kernel, auth, mail, etc.)
Severities = HOW serious (debug → emerg)
Rules = WHERE messages go (based on facility + severity)
Rotation = HOW logs are managed (prevent disk exhaustion)
Troubleshooting = Verify daemon → check config → find log → grep → test

