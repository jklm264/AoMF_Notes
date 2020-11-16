# Linux Networking: Tunneling

- A way for a remote network or user to connect to a private network
- Authentication via creds or certs
- enter VPN

## Virtual Private Networks (VPN)

**Full Tunnel** - all traffic (ingress/egress) goes thru tunnel

**Split Tunnel** - traffic bound for internal/local resources goes over tunnel and is encrypted. All else goes through normal/non-VPN connection

- Ex: If network is 192.168.2.0/24:
  - Hxxps://www.google[dot]com **would not** go through the VPN
  - Hxxps://19.2168.2.50/sharepoint **would** go through the VPN
- Also [internet connection sharing](https://answers.microsoft.com/en-us/windows/forum/windows_10-networking-winpc/internet-connection-sharing-in-windows-10/f6dcac4b-5203-4c98-8cf2-dcac86d98fb9)

### VPN Protocols

**IPSec** - strongest encryption with certs

- originally developed for IPv6
- Seen on port security so others can't plug in to your network.
- **Transport Mode** - encrypt payload only. IP heead is not modified or encrypted
- **Tunnel Mode** - entire packet is auth'd and encrypted with new IP and header

**PPTP** - Uses TCP over a GRE tunnel to encapsulate PPP packets

- Point-to-Point protocol used in dial-up
  - Asynchronous PPP
    - Acts as virtual network interface
    - Used in Dial-up
    - Adds link-level headers and separates packets
  - Synchronous PPP
    - Encapsulation protocol used in high-speed circuits
    - More common
    - Can run over etc, primarily in DSL/cabel lines

**SSL** - for web browsers

- can do [**stunnel**](https://www.stunnel.org/) 
  - packets set over the tunnel are encrypted via ssl certs
  - transport unencrypted logs to a central server



## SSH Tunnels

- replaced tenet as secure remote access.
- Can also port forward/tunnel, scp, doesn't require special vpn server



**Create a secure tunnel for web browsing**: `ssh -D 8080 -N user@sshserver` then set the web browser to proxy on *localhost:8080*



## X11 Forwarding over SSH and Remote Desktop

X11 = the windows manager ("graphic engine")

- SSH allows to connect not just to terminal (tty) but X11 server too!
- Could forward displays to other computers to remote desktop
  - Would need to enable X11 forwarding on both host and client/target in `/etc/sshd.conf`.
  - Uses SSH to securely forrward X11 and encrypt (and authenticate with a cookie) these comms
  - Can do this via Linux-to-Linux and Windows-to-Linux
  - Ex: Could run firefox on target machine and would show up on host machine- **NOT** on the target machine. **Simply exporting the display**
    - 'X-Aware' - can transfer a pseudo-display



Can do this via: `$ssh -v -X user@hostname`

- can have x11 by default by enabling X11 by default in config files



### Alternatives

- NX
  - Can serve windows, linux, or Mac
  - `yum install nx freenx; nxserver --adduser <user>; nxserver --passwd <user>`
  - *[FreeNX](https://wiki.archlinux.org/index.php/FreeNX)*
- VNC
  - Typically slower than X11 forwarding
  - typeVNC
  - realVNC



##Kernel Tunneling DEMO

**To see which can be tuned**: `$sysctl -a`



Edit `/etc/sysctl.conf`

**To protect**:

- *net.ipv4.conf.default.rp_filter = 1* - verify source route to protect against IP spoofing
- *net.ipv4.tcp_syncookies = 1* - protects from syn-flood attack (prevents multiple connections if 3-way handshake is never completed)
- *net.ipv4.ip_forward = 0* # no ip forwarding on behalf of other devices: TUNNELS!
- *kernel.ctrl-alt-del = 1* # disallows users to ctrl+alt+delete from command line
- *net.ipv4.icmp_echo_ignore_all = 1* # hides from ping sweep by ignoring all icmp traffic

#