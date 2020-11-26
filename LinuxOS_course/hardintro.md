# Linux Hardening

- Use security-enhancing software like iptables, SELinux, AIDE.

- Remove/Modify user accounts as necessary.
- Segregate services onto difference boxes.
- Store logs offsite.
- Host.allow
- automate update schedule
- Kernel parameters

#### Specific Hardening

- To figure out what services are enabled at boot time, run the command: `/sbin/chkconfig –list` (shows run-levels; recall 5 is standard logon)

- Remove X11: `yum group remove "X Window System"`

- Disable IPv6: In `/etc/modprobe.conf`

  ```bash
  alias net-pf-10 off
  alias ipv6 off
  ```

  then in `/etc/sysconfig/network` then reboot.

  ```bash
  NETWORKING_IPV6=no
  IPV6INIT=no
  ```

- Find and disable SUID/SGID

```bash
find / \( -perm -4000 -o -perm -2000 \) –print
chmod -s filename # see Linux Hardening Guide for specifics
```

- Disable services if you don't need them:
  - anacron apmd, hidd, microcode_ctl, autofs, pcscd, readahead_early,readahead_later, bluetooth, kdump, kudzu, gpm, mdmonitor, hplip, avahi-daemon, cups, rhnsd
  - `chkconfig service_name off`
- SSH configuration
  - In `/etc/ssh/sshd_config`

```bash
AllowGroups group #only allow specific upsergroups
PermitRootLogin no
Protocol 2 # For SSHv2 only
```

- Kernel Parameters in `/etc/sysctl.conf`

```bash
net.ipv4.conf.all.rp_filter = 1 # Enable IP spoof protection using reverse path filter
net.ipv4.conf.all.accept_source_route = 0 # Disable IP source routing
net.ipv4.tcp_syncookies = 1 # Preventing a SYN flood
net.ipv4.conf.all.accept_redirects = 0 # Disable ICMPredirect acceptance
```

