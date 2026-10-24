# smb_mount

This role is a helper to configure a read only smb mount to `smb_mount_target_dir` (the local mount point).

All connection parameters are asked for interactively:

* `CIFS Sharename: the remote sharename, e.g. `//myfileserver/music`
* `username`: The user that is used to connect to the SMB Server
* `domain`: The smb domain that user belongs to
* `password`. The password of that user. It is not echoed during input.

The role
* creates a file containing the mount credentials at `/etc/music_mount.creds`
* adds an entry to `/etc/fstab`
* mounts the the directory via `/etc/fstab`. This is not really the ansible way, but even when `ansible.posix.mount` supports both mounting and managing the entry in `/etc/fstab`, it does **not** use the entry in `/etc/fstab` to actually do the mount.

Idempotency is ensured by checking if there is an `smb3` mount on `smb_mount_target_dir` before starting the interactive parameter queries. So normally, only on the very first run you will have to enter these parameters.

## Variables

* `smb_mount_target_dir` defaults to `/srv/music` as defined in the [defaults](defaults/main.yml).
