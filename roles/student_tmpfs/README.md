# student_tmpfs

Ansible role that configures an ephemeral, tmpfs-backed user account with USB mass-storage blocking.

## Variables

| Variable | Default | Description |
|---|---|---|
| `student_user` | Derived from `inventory_hostname` (`student-pc-01` → `student_01`) | Username to create |
| `tmpfs_size` | `512M` | Size of the tmpfs home mount |

## What it does

1. **Create student user** — standard Linux user with a home directory.
2. **Mount tmpfs on student home** — replaces the on-disk home with a RAM-backed tmpfs mount (`mode=0700`, isolated per-user). Nothing written here survives a reboot.
3. **Deploy home-init systemd unit** — a oneshot service (`<user>-home-init.service`) that runs before the display manager on every boot, copying `/etc/skel` into the fresh tmpfs home and fixing ownership. This is what gives the user their "blank slate" on each boot.
4. **Deploy USB mass-storage block** — a udev rule denying `bInterfaceClass=08` (mass storage) devices. Keyboards/mice (HID class) are unaffected.
5. **Enable home-init service** — ensures the unit runs on every future boot.

## Why tmpfs + systemd instead of just cleanup scripts

- tmpfs guarantees data cannot survive past `umount`/reboot — no cleanup script can fail to catch an edge case, because the storage itself is volatile.
- Ordering the init service `Before=display-manager.service` guarantees a clean home exists before any user can log in — no race condition.

## Testing

Manually verified: write a file as the student user, reboot, confirm it's gone and `/etc/skel` contents are restored. See main repo README for full validation notes.
