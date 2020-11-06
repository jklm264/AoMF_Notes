# User C(R)UD

## Adding Users

Recall:

- `#useradd` check *-D* for defaults, will not create new home directory nor make user's home shell
- `#adduser` automates! Will create new home directory

To modify /etc/{passwd,shadow}: `#vipw` - will lock /etc/shadow so no race conditions.

### Defaults for New Users

- startup scripts are in skeleton in `/etc/skel`
  - `/etc/profile` is system-wide config

### Adding Bulk Users

- With `$newusers <input file>`; input file requires cleartext passwords 

### Reseting Passwords

Use `$passwd`

---

## Modifying a User 

- `$usermod` can change home direectory, shell, expiration date of password, GID

---

## Deleting Users

- Automatically with `$userdel`
- Manually remove:
  - Alias files, crontab, home directory, entries in /etc/passwd, /etc/shadow, /etc/group.
- Make sure no files or processes are running in memory with: `$find / -xdev -nouser` #This will find it, not delete. Will still have to `$kill` them.

### Temporarily Disable an Account

- Use `$usermod -L <username>` will lock the account.
  - This just puts a '!' in front of the encrypted password in `/etc/shadow`.
- To unlock, `$usermod -U <username>`

