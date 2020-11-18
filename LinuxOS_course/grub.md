# Linux Security Features - GRUB

Grand Unified Boot Loader - replaces Master Boot Record

- For more, see Jordan's Week 5A



**GRUB Legacy**

- Used by RHEL/CentOS and SUSE

- Config: `/boot/grub/menu.lst`
  - Can change boot order and timeout time
  - splashimage is graphic during boot
  - hiddenmenu - shows full adv options during boot before booting default
  - **vmlinuz** is a compressed **Linux kernel** [src](http://www.linfo.org/vmlinuz.html#:~:text=vmlinuz is a compressed Linux,compressed and non-bootable form.)
- Allows Single-user mode access ([See *recovery mode*](Recovery.md))
  - Secure boot with password; Uses md5
    - $grub-md5-crypt
    - Paste hash of password into `menu.lst` with *lock password --md5 <hash>*
  - Pro: secures from Single-User mode
  - Con: Can still be overcome by booting off of a live cd/usb and simply mount file system for access.
    - Need to have BIOS protection and do NOT allow boot from CDs/USBs/external devices
  - Con: During system restart, need to enter GRUB password every time.



**GRUBv2**

- Used by Fedora and Ubuntu
- Config: `/boot/grub/grub.cfg`
  - grub.cfg is dynamically generated via `/etc/grub.d/*` and editted using *#update-grub*
    - 00_headeer - reeads /etc/default/grub. Stores GRUB user/passwords
    - 10_linux - loads menu entries for **default installation**
    - 20_xxx(windows-7) - other distros
    - 30_os-prober - scan hard disks for other OS's
    - 40_custom - creeates additional entries to boot menu
- `/etc/default/grub` contains menu settings with *GRUB_DEFAULT* and *GRUB_TIMEOUT*
  - Also theme customization; 'make bootup pretty'
- Setting a password in GRUBv2
  - Auth user must be identified
  - passwords for the users must be delegated
  - Menu entirees are modified with usernames/passwords (seperate from users on the actual OS [ex: /home/user/bob])
  - E.g. If we only want userA to boot to windows 7 partition we could add their account to the 21_windows-7 menu item.



## GRUB Demo

**To restrict access**

- \#grub-md5-crypt; # enter password and copy hash
- \#vim /boot/grub/menu.lst
  - At end of partition, `password --md5 <password_hash> lock` #Lock will make sure system can't be booted without password input required.

