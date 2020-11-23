# Linux Host Intrusion Detection System
Purpose: Monitor hosts for evil.

- Monitor file/directory changes
- Monitory system activity/logs
- Report on evil found

## Adv Intrusion Detection Engine (AIDE)
 - File/directory integrity checker (same as [tripwire](https://github.com/Tripwire/tripwire-open-source))
 - Needs to be run as cronjob; so no 'real-time' protection.
 - Suggested to take db of hashes on a gold image off current machine.   
 - Config: `/etc/aide.conf`
 - **Attribute checks:** File type, Permissions, Inode,UID,GID,Link name,Size,Block count,Number of links,Mtime, Ctime, and Atime 
 - Default protected dirs: */boot,/bin,/sbin,/lib*

## Open-Source IDS (OSSEC)
- [OSSEC site](https://www.ossec.net/)
- Uses agents for *real-time* alerting/protection
- Has file integrity checking, log monitoring, rootkit detection (parttern-based), active response (heuristic-based)

