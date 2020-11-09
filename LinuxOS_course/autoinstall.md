# Linux Automated Installations

RHEL uses Kickstart app to automate installs

- Uses Anaconda for scripting interface
- Like Kickscripts in Windows? (Idk)

- Kickstart config file sections
  - Command: Keyboard, Language, URL for package install files, root password, etc.
  - Package Installation
  - Shell Commands

**1. OpenSUSE AutoYaST**

- GUI 3 phrase

**2. Ubuntu Debian Installer**

- Will be a clean-wipe installation
- Uses an "answer file"



## Managing Packages

- Red Hat Package Manager (RPMs)
  - fedora, centos, etc
  - To access use `$rpm`
  - Yum is it's repository manager
    - Note, repositories are not responsible for dependacies 
- Debian (.deb)
  - debian, ubuntu, linux mint, etc
  - To access use `$dpkg`
- Can switch between the two using [alien](https://joeyh.name/code/alien/)



## High Level Package Management

Package mgmt with pure packages 1) don't include dependancies, 2) have broken installations, 3) might not download latest version (by default).

- So we have metapackage mgmt systems (apt, yum, red hat network)
  - These repositories will install the dependencies for you and maintain them.

### Repositories (Repos) Demo

- Repositories are a collection of packages
  - Use GPG signature signing for checksums
    - `$rpm --checksign PACKAGE_NAME`

**Advanced Package Tool (APT)**: 

- Debian-based distro's 

- Ruby-based
- `/etc/apt/sources.list`
  - deb for package repository (binaries only) and deb-src for source file repository (source code)
  - repository URI (can also point to somewhere on disk)
  - Distribution
  - Component(s)
- To update: `$apt update && apt upgrade -y`

**Yellowdog Updater Modified (Yum)**: 

- Red Hat type distro's like CentOS, Fedora, Scientific Linux, yellowdog linux

- `/etc/yum.conf` OR `/etc/yum.repos.d/yum.conf` configuration settings and can specify repo's here, though most do it in the next one.
  - "logfile=*", "gpgcheck=1"
- `/etc/yum.repos.d/` stores file and specifies repo's
  - In *CentOS-Base.repo*: "mirrorlist=\*","gpgkey=\*"
- To update: `$yum update`



## Repos and System Mgmt

**Custom Repositories**

- FTP or HTTP server with `$createrepo` [src](https://linux.die.net/man/8/createrepo)

**Tools**

- Red Hat Network Satellite $$$
  - Configure 1000s of systems at once
  - schedule updates or maintenance
  - Web-based interface
  - Red Hat Spacewalk (Free version)



