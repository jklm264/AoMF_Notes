# Other Linux Threats and Vulnerability Techniques

## Misconfigurations

"Users are the weakest link in security" - Jeff Arsenault 

- Default Passwords/Giving out root password
- Accounts without Passwords (`cat /ect/group | awk '{print $1 $2}'`)
- Setup file logs (Solaris - delete all files in tmp on shutdown)
- Shared files (misconfigured NFS **OR** *ugo* has RW access)
- Open ports that allow admin access (web server mgmt like mysql)
- No GRUB password
- Using SSHv1
- Using Telnet
- Not doing least priv (many users have sudo)



## Software Vulns

- Buffer Overflows - Mismanagment of memory by a program
- Input validation - XXS, SQLi, RCE
- Drive-by downloads - via Java (since it's cross platform)

**Mitigation**: Patch and test own development projects

- If you can't patch immediate, suggested to disable to software until you are able to upgrade.



## Social Engineering

See [my OSINT notes](https://github.com/jklm264/My-Forensics-Notes/tree/master/OSINT_Forensics)

#