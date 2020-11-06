# Linux File System Security

- Parition:= Logical set of objects
- Built in Logical Volume Groups (aka partitions)
  - As shares (NFS, CFS, disk-based storage)
  - Usually mount at startup

Mounting

- mount <src> <dest>
- sda1, sda2 - numbers are partition number
- mount simply links the FS to running OS
- unmounting
  - umount
  - fuser -c := can see what user is accessing a certain file/directory

## Secure Mount Points

- Everyone has access to /tmp, /var/tmp, /dev/shm
- Need to put W/X permissions
- /etc/fstab - shows where a partition (with permissions) will be mounted at startup
  - nodev, noexec, **nosuid**
- *To secure mount points*, you can add lines in the fstab file
  - Ex: `/dev/<src> <dest> ext4 defaults,nodev,nosuid,noexec,bind,rw 1 2`
  - Usage: <dev>  <mount point> <fs-type> <options> <backup operations><fs check order>
    - Backup operation: 1 if dump utility should back up a partition, 0 if it shouldn't
    - File system check order: $fsck checks for errors at boot time. 0 means should not check. Higher number represents check order. Root parition should have 1; all else should be 2.
  - Note, will need to reboot for this to take effect. However you may use $mount's options for on-the-fly changes.

## File Security

- Permissions by user/owner, group, world
  - R=4, W=2, X=1
  - Use `$chmod` to change permissions 
  - Chown and chgrp
  - setuid & setguid (set user/group ID)- temporarily elevated privileges in order to perform a specific task if UID or GID are met. 
    - Use chmod u+s OR **4**777
    - User: u+s OR **4**755
    - Group: g+s OR **2**755
  - sticky bit - so only the file's owner, the directory's owner, or root user can rename or delete the file
    - ls -ld will show *d* at end of permission list
    - +t OR **1**755
  - chroot - changes the apparent root directory for the current running process and their children ("linux jail")
- When new user created, they are created with own group
  - `#useradd` check *-D* for defaults, will not create new home directory nor make user's home shell
  - `#adduser` will create new home directory
  - Can confirm in /etc/passwd and /etc/group

### File Inodes

- File has 2 parts: data block and metadata (aka inode)
- Inode contains: time/data creations, time/data last modify, file size, link count, permissions
  - If exec permission on directory means directory may be copied or access by all users



## File Access Control 

### Discretionary Access Control (DAC)

Linux ACLs that have to be setup per file, per machine; aka no centralized policy. 

- Con: Software is trusted (meaning policy does not care what the program is or does, just checks permissions)
- Con: All programs that the user has access to, the programs also have access to all that user's files

#### Access Control List

- Least Privilege 

- To set access: `$setfacl -m u:nginx:rx /home/user1` #Allow nginx to access user1's directories 
  - Group access *g:<group>:rx*
- To check if there's access, look for '+' sign at end of `ls -al`
- To list access: `$getfacl <file/dir>`
- To remove access: `$setfacl -x u:nginx /home/user1`
- *Most restrictive permissions apply first*



### Mandatory Access Control (MAC)

- Pro: Centralized management of configuration

- Pro: Distruste everything
- Pro: Programs can be contained
- Con: Very complicated

Ex: LIDS (protects even from root), Tomoyo (very user friendly), SELinux (NSA-made)



## AIDE

- program that creates a database of all metadata for a file
- When `$aide --check` is run, it will output all metadata changes per directory, and per file.



## Kernel Tuning

- In `/etc/sysctl.conf`
  - prevent syn flood: `net.ipv4.tccp_syncookies = 1`
  - disable IP source routing (so you can't be used as a router): `net.ipv4.conf.all.accept_source_route=0`
  - disable icmp reditect acceptance: `net.ipv4.conf.all.acccept_redirects=0`
  - disable ping requests: `net.ipv4.icmp_echo_ignore_all = 1`
  - Others: set max shared memory, set anode-max, enable ip spoof protection, disable ipv6



## Linux Drivers

- Can add custom linux loadable module; theese may have vulnerabilities



## Host Access Controls (HAC)

- Network firewalls

- TCP Wrappers: /etc/hosts.allow and /etc/hosts.deny

  - Utilized by inetd, xinetd, sshd

- For hardening, you would deny all and allow few (whitelist method)

  - Ex: In /etc/hosts.allow: ` sshd: 	10.0.0.6/255.255.255.0`
    - Then in /etc/hosts.deny: ALL: ALL`

- Also [Fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page)

  









[NSA Linux RHEL5  Hardening Guide](https://apps.nsa.gov/iaarchive/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm)
