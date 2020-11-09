# Linux Viriuses and Worms

"It's common user has admin privileges (sudo group)"



## History of Linux Viruses

1996 - Staog: First real linux virius; kernel flaw via an executable

2001 - Ramen; attacheed to zips and replaced index.html within them

2002 - Mighty and Lupper worm; open IRC channeel and wait for commands

2010 - Koobface virus; Spread through social networking sites and gain passwords then send infected messages to friends


## Trojan Horses

- Usually uncovered by the software developer

- Make sure checksums match

### History

- Usenet (BBS) had the trojan Turkey that would draw a turkey on the screen and then delete your home directory.

- 2011 - Kernel.org's OpenSSH binary was modified to look at user activity logs.
- 2012 - Sourceforge.net's mirror site's phpMyAdmin was replaced with a malicious one that had a backdoor.



## Rootkits

Def: Program that hide malicious mactivities from the system and cover their tracks to avoid detection. Hard to detect since it's usually put in with a lot of 'normal' programs. Will use system calls to intercept and replace commands.

- Most famous: [Sony CD rootkit for DRM](https://en.wikipedia.org/wiki/Sony_BMG_copy_protection_rootkit_scandal)

- Can find all 233 linux rootkits (since Feb 2013) at PacketStormSecurity.com

**To get rid of rootkit:** Get your critical data off the system --> wipe the system --> start from scratch.

### Homemade Rootkit

Write over `ps, ls, netstat, find, inetd, login, and lsof` to hide rootkit process