# Linux Networking

## IPv4

127.0.0.1 loopback ip/device

**IPv6**

- 16 bytes address space
- Pro: NAT and DHCP is not necessary
- **DoD has required support in any new products (starting 2013) it purchases**
- Disable IPv6: `echo net.ipv6.conf.all.disable)ipv6 = 1" >> /etc/sysctl.conf`

## Network Manager

- easily configure networking the devices without going into multiple text files and editing by hand
- Usually GUI

### /etc/hosts File

- Tells system how to translate a name to an IP address
- **Overides anything in/before-searching DNS**

### /etc/resolv.conf File

- Stores dns server and local search domain



**RHEL-specific**: GUI or `$system-config-network` (TUI) or CLI

- `/etc/sysconfig/network`: hostname, default route
- `/etc/sysconfig/static-routes`: static routes  
  - Ex: `eth0 net 10.10.10.0 netmask 255.255.255.0 gw 10.10.10.1` 
  - Or use `$route-add`
- `/etc/sysconfig/network-scripts/<ifcfg-interfacename>`: specific iface configs 



**Ubuntu-specific**: `/etc/network/interfaces`



## Wireless

- use a VPN





## IPtables

- basic firewall

## DNS(bind)

- DNS Linux kernel software

##

