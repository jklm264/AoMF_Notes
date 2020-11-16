# Linux File Sharing

NFS - Network File Sharing

On server, NFS server *exports* (directories)

Pros:

- stateful file ops
  - **NFS State**:= what files are accesseed, locked, query post-crash
- integrated security
- ACL's



**NFS Security**

- Uses *rpcsec_gss* (vs auth_sys and auth_none)

- 2049/TCP and 2049/UDP and 111/TCP

- `/etc/exports`

  - **Set export permissions from this file; permissions on individual dirs/files won't affect the *export* permissions**
  - *root_squash* doesn't allow root access
  - *no_root_squash* does allow root access

  

### On NFS Client

Need to mount yourself:`$mount -t nfs4 -o intr,bg server:/nfs/mountpoint`

- BG - if server fails, keep trying mount in background
- Intr - allows user to interrupt blocked operations



## Samaba

- Works with AD and Windows shares
- Developed by Common Internet File System (CIFS)
- CIFS basic services
  - authenticate/authorize
  - printing
  - service announcements
  - stand-alone, watered-down active directory
  - File share



**Config File**: `/etc/samba.smb.conf`

