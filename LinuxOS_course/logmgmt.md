# Linux Log Management

## Syslog
- Help programmers to allow for centralize logs
- Syslogd - baseed on `/etc/syslog.conf`
- Redhat now calls it `/etc/rsyslog.conf` (now multithreaded)
- Can set retention policies though suggested logs be moved off of machine

## Other Tools/Locations
1. System Messages - `/var/log/messages`
2. Configuration Files - `/var/log/`
3. lastlog - last login times (wtmp)
4. wtmp - not readable
5. `/var/log/secure` - indiv user sudo activity
6. yum.log - installation logs
7. `/var/log/audit.log`

