# SELinux

**Security Enhanced Linux**

- Developed by NSA
- Integral in RH-based distros; enabled (not enforced) by default since RHv4
- Mandatory Access Control for Linux - enforces rules on files, processes, and their actions based on policies.

**Config:** `/etc/selinux/config` or GUI (*$yum install policycoreutils-gui*)

States:

- enforcing - enabled, will block
- Permissive - prints warning, but won't block (great for testing!)
- Disabled - no policies loaded

Policy Types:

- Targeted - enforce on specific services
- Strict - enforce all policies (usually you don't do)

File Attributes: SELinux User, Role, Type, Level



Example: Deny users to have RW to their own FTP/SFTP

- `$getsebool ftp_home_dir` #By default is off
- `$setsebool -P ftp_home_dir on` # Allows RW access now



**Listing current SELinux security contexts**: `$ls -alZ`

