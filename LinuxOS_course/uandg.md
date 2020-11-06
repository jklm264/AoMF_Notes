# Users and Groups

To list 'em use `/etc/passwd` file as best bet.

- Maps UID to username
- Can be replaced with directory service solutions like OpenLDAP, Kerberos, Active Directory, Fedora Directory server, etc.

When a user logs in, they use the conninical username, not UID. Linux will lookup a user's shell preeference, permissions, and more all from the /etc/password file



BONUS: How to lock or disable an account, [see here](https://www.thegeekdiary.com/unix-linux-how-to-lock-or-disable-an-user-account/).

## Passwords

In `/etc/passwd`, if the second field (password) has an 'X' it means the user has a password. If this field contains a '+' or '-', it means the passwords are stored on another machine as the environment is using a directory service (examples listed above). If this field is empty, this denotes this account does not have a password.

A user's actual password is stored in the `/etc/shadow` file where the password is encrypted. Encryption may be figured out based on how the password is formatted. [See here](https://hashcat.net/wiki/doku.php?id=example_hashes) for hash examples.

- Login names are typically <32 chars [a-Z,0-9]

- **The default Linux encryption of passwords is DES**
  - DES < MD5 < SHA1/Blowfish
  - May be changed in `/etc/login.defs`

### Users

- root is UID 0
- Real users have UID >500

### Groups

- Look to `/etc/group` for more info

### GECOS Field in /etc/passwd

Where user description is located: Full name, office number and building, office phone extension, home phone numbers

### Shells in /etc/passwd

- `sbin/nologin` or `/bin/false` - where user is not allowed to login

  

