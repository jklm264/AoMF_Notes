# Linux Anti-Virus
Consider, your linux file-hosting/mail server holds Windows executables that may be pulled by any user on any box at any time. You want AV on Linux in case their AV's won't catch it.


Rkhunter - [easy instructions](https://www.tecmint.com/install-rootkit-hunter-scan-for-rootkits-backdoors-in-linux/); uses integrity check mainly.
Chrootkit - [easy instructions](https://www.redhat.com/sysadmin/3-antimalware-solutions); uses LOLBins (strings, grep, etc)

**The standard Linux AV**: ClamAV - [see more](https://www.digitalocean.com/community/tutorials/how-to-setup-exim-spamassassin-clamd-and-dovecot-on-an-arch-linux-vps)

