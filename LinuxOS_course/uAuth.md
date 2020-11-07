# User Authentication

## PAM (Pluggable Authentication Modules)

- Centralizes authentication through shared libbraries for porgrams like `login, sudo, passwd, and su`
- Custom modules like password complexity, retry-until-banned, and MFA
- `/etc/pam.d` where each line is: *module-type control-flag module-path arguments*
  - Module-type: 
    - Auth (identify the user)
    - Account (enforce restrictions such as number of simultaneous logins)
    - Session (tasks before or after a user logs in)
    - Password (change a user's password)
  - Control-flags: signal-like. 
    - Required – failure will eventually cause the stack to fail
    - Requisite - fails the stack immediately

[BONUS] Kerberos completely bypasses PAM, unless using pam_krb5 module



## Centralized account management

- Lightweight Directory Access Protocol (LDAP) - db that centralizes UIDs and GIDs to manager uname's and passwords for authentication.
  - OpenLDAP
  - Con: Sends password in cleartext!
  - Con: Need network access else can't get authentication query out
- Active Directory (AD)
  - Uses samba as well as Microsoft's implementation of Kerberos

We like Single-sign-on