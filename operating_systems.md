# Choose the Right Operating System

Vultr offers different Operating Systems (OSs) as templates. Some are better suited than others for specific purposes. A feature which is an advantage for one user can be a disadvantage for another user.

## [Alma Linux](https://www.vultr.com/servers/alma-linux/)

Alma Linux is an open-source Linux based on Red Hat Enterprise Linux (RHEL). In 2020, Red Hat stopped supporting CentOS as a downstream fork of RHEL. The CloudLinux company created Alma Linux as a CentOS alternative. The first stable release was published in 2021. Alma Linux is 1:1 binary compatible with the latest RHEL. It is maintained by a non-profit entity funded by CloudLinux and other companies.

### Pros

* Viable alternative for CentOS
* Many useful features for enterprise use-cases
* Lean operating system with no bloatware
* Works well for high availability clusters
* Native packages for Distributed Replicated Block Device (DRBD) storage systems
* Focus on stability

### Cons

* Sometimes ships with outdated packages
* Not suited to be used as a desktop OS
* Not suitable for novice users
* Lacks robust driver support for niche hardware devices (this is not a concern while installing from a Vultr template)
* Configuring network interfaces can be cumbersome
* New OS with a smaller userbase

## [Arch Linux](https://www.vultr.com/servers/archlinux/)

Arch Linux is an independently built Linux distribution. It allows to configure and customize every part of the OS.

### Pros

* Allows to build a completely customized OS
* Zero bloatware
* Continuous updates with rolling release model
* Complete control over every part of the system
* Excellent hardware support for different devices and interfaces
* Secure configuration by default
* Learning opportunity to know the inner workings of the OS

### Cons

* Bad choice for a novice
* No graphical installer bundled with the system
* Elaborate installation procedure, requiring specialist knowledge in choosing and configuring necessary packages
* Works only on x86_64 architecture
* Rolling updates mean that some updates can break the system
* Upgrades often need manual intervention to get it right

## [CentOS](https://www.vultr.com/servers/centos/)

CentOS is (was) a downstream rebuild of RHEL.

Note: The RHEL ecosystem has a few different distributions. Fedora is the upstream. New changes are first made in Fedora, which has the latest software. From Fedora, changes come to CentOS Stream. After CentOS Stream, changes are reflected in RHEL. CentOS is (was) downstream from RHEL, and hence the most stable. In 2020, Red Hat stopped its support of CentOS. Successors to CentOS include Alma Linux and Rocky Linux.

### Pros

* Many useful features for enterprise use-cases
* Strong track record as a server OS
* Lean system with no bloatware
* Works well for high availability clusters
* Native packages for Distributed Replicated Block Device (DRBD) storage systems
* Focus on stability

### Cons

* Sometimes ships with outdated packages
* Not the best choice for a desktop operating system
* Not suitable for novice users
* Lacks robust driver support for niche hardware devices (this is not a concern while installing from a Vultr template)
* Configuring network interfaces can be cumbersome
* Project is no longer active

## [Debian](https://www.vultr.com/servers/debian/)

Debian is a stable and mature Linux with a large ecosystem. It is the base system for many popular distributions like Ubuntu, Kali Linux, Knoppix, Element OS, Linux Mint, etc.

### Pros

* Stable and reliable
* Packages with reasonable default configurations
* Regular security updates to packages
* Extensive hardware support, including proprietary drivers where necessary
* Works on most desktop and server hardware
* Supports different CPU architectures (e.g., x64, i386, ARM and MIPS, POWER7, POWER8, IBM System z and RISC-V)
* Runs on wide variety of devices
* Flexible installer suitable for both new users as well as experts
* Large number of available packages (55000+)
* Long Term Support (LTS) version with at least 5 years of support and upgrades

### Cons

* Because of the focus on stability, it sometimes ships with outdated packages. This can be problematic if your application needs the latest software. You might be able to compile newer packages from their source code.
* While it is suited for desktop use (owing to stability and hardware support), the default interface feels dated. This can be solved by installing a modern GUI.

## [Fedora](https://www.vultr.com/servers/fedora/)

Fedora (sponsored by Red Hat) is the upstream distribution of RHEL.

### Pros

* Speed of new updates, access to the latest software
* Different editions for different uses (Fedora IoT for IoT devices, Fedora WorkStation for developers, Fedora Silverblue for the desktop)
* Pre-packaged configuration options (Fedora Spins allows you to choose a different desktop environment like KDE or XFCE while Fedora Labs has curated collections of domain-specific software packages)
* Suitable for casual desktop use

### Cons

* Rapid integration of new software can sometimes lead to instability
* Not the best choice for network intensive tasks
* Not the best environment for sysadmin tasks

## [Fedora CoreOS](https://www.vultr.com/servers/fedora-coreos/)

Fedora CoreOS is a minimal OS designed for running (hosting) containers at scale. It was designed to be spun up as needed by new Kubernetes instances. Fedora CoreOS was created when Red Hat acquired Container Linux.

### Pros

* Lightweight
* Suitable for use as a container host
* Automatic upgrades over multiple machines
* Atomic updates which can be rolled back
* Security: minimal OS implies a narrow surface of attack

### Cons

* Not the right choice for general purpose workloads
* Not suitable to run as a guest OS inside a container
* Does not come with a package manager
* New OS with a smaller community (the first public preview was released in 2020)
* Breaking changes during the transition from Container Linux to CoreOS

## [FreeBSD](https://www.vultr.com/servers/freebsd/)

FreeBSD is an advanced Unix-like OS for servers. It is a monolithic OS. The kernel, device drivers, userland applications (system software) are all delivered by the same project. It forms the basis of much of the codebase of Darwin, and hence the Apple family of operating systems (MacOS, iOS, etc.).

### Pros

* Predictable and stable system
* Strong network stack, leading to better performance for networking applications
* Monolithic system leading to better integration of userland tools
* Well-documented and clean code
* Single repository for kernel and userland code
* Comes with sensible defaults
* Includes DTrace - to collect real-time system statistics with negligible overhead
* Includes ZFS - an advanced file system focused on data integrity and large-scale storage

### Cons

* Smaller userbase compared to Linux
* Limited (though active and high quality) online forums for seeking support
* Some features may be overkill for new sysadmins
* Limited hardware and driver support for desktop use

## [OpenBSD](https://www.vultr.com/servers/openbsd/)

OpenBSD is another BSD variant with a focus on security and stability. It is the upstream source of many well-known packages like OpenSSH, LibreSSL, PF, etc.

### Pros

* Predictable and stable operating system
* Suitable for high security environments
* Strong network stack, leading to better performance for networking applications, e.g., router, firewall, DNS server, etc.
* Monolithic system leading to better integration of userland tools
* Well-documented and clean code
* Single repository for kernel and userland code

### Cons

* Small userbase and online community
* Not suitable for inexperienced users
* Used for specific niche applications
* Userland tools are often outdated
* Not the best for general purpose server use

## [Rocky Linux](https://www.vultr.com/servers/rocky-linux/)

Rocky Linux is an open-source Linux designed to be 100% compatible (including bugs) with RHEL. It was started by one of the founders of CentOS after Red Hat discontinued their support of CentOS. Rocky Linux is supported and sponsored by Google Cloud.

### Pros

* Viable alternative for CentOS
* Many useful features for enterprises
* Lean operating system with no bloatware
* Works well for high availability clusters
* Native packages for Distributed Replicated Block Device (DRBD) storage systems
* Focus on stability

### Cons

* Sometimes ships with outdated packages
* Not suited to be used as a desktop operating system
* Not suitable for novice users
* Lacks robust driver support for niche hardware devices (this is not a concern while installing from a Vultr template)
* Configuring network interfaces can be cumbersome
* New OS with a smaller userbase

## [Ubuntu](https://www.vultr.com/servers/ubuntu/)

Ubuntu is the most popular Linux, both for desktop and servers.

### Pros

* Large online community (makes it easy to get support)
* Possibility to get commercial support from Canonical
* Used at large companies like Google
* Better than many other Linuxes for day-to-day desktop use
* Large number of packages for different applications
* Easier to find sysadmins with Ubuntu experience
* Desktop experience with the OS can be helpful while working on the server

### Cons

* Less stable than Debian
* Breaking changes are sometimes introduced in the upgrade process
* Larger attack surface because of larger number of preinstalled applications
* Not all packages are actively maintained

## [VzLinux](https://www.vultr.com/servers/vzlinux/)

VzLinux is based on the RHEL source code. It is maintained by Virtuozzo, a Swiss company specializing in virtualization software.

### Pros

* Aims to be 1:1 compatible with RHEL
* Stability is a key goal
* 20+ years track record as the base OS of virtualization products from Virtuozzo
* Optimized to run in containers and virtual machines
* Includes tools to convert from CentOS, and to do dry runs before converting

### Cons

* Smaller user base compared to other distros
* Some packages are outdated
* Uses a dated GUI

## [Windows Server](https://www.vultr.com/servers/windows/)

Windows Server is made by Microsoft. It does not include consumer features like Cortana. It includes many server-side applications and is configured with tight security settings. Windows Server can handle heavier hardware than Windows. By default, it runs on the command line; installing the Windows GUI is optional.

### Pros

* Wide range of software choices
* Easier to deal with different versions
* Possible to add on the Windows GUI and get a similar user experience as Windows desktop
* Easier for casual users to run and manage owing to familiar Windows GUI
* Compatible with most kinds of hardware
* Cheaper and easier to find sysadmins
* Easier to get support online because of ubiquity of Windows
* Excellent domain management tools make it easy to centrally manage a large number of PCs
* Long term support by Microsoft

### Cons

* Windows Server licenses are expensive and restrictive
* Needs more hardware (compared to Linux and Unix based systems) to run the OS
* Large number of preinstalled applications which may not be necessary
* Privacy concerns
* Security: larger surface area of attack due to larger number of preinstalled applications
* Not the best choice for running scalable webservers
* Not possible to customize the (closed source) OS
* Subpar developer experience compared to Linux and Unix based systems

## [Windows Core](https://www.vultr.com/servers/windowscore/)

Windows Core is a bare bones operating system. It does not include any proprietary binary-blobs or software. Because of this, the platform could be made open-source. It is not designed for consumers. Windows Core is a "modular" platform. Device manufacturers and developers can build custom Windows operating systems for their devices.

### Pros

* Smaller and less resource hungry than a normal Windows Server
* Only has the core OS elements which are necessary to run server-side applications
* Suitable for running in Docker environments
* Fewer default applications (this makes it more secure due to a reduced attack surface)
* PowerShell scripts are compatible across Windows Server and Windows Core

### Cons

* Doesn’t run all the processes necessary for a normal Windows installation
* Not possible to use as a regular desktop OS
* No Support for printing, plug and play hardware, sound, graphics, etc.
* Command line only (this can be challenging for new users)
* Not as popular as Windows Server
