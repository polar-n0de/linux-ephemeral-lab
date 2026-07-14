# linux-ephemeral-lab

Ephemeral, tmpfs-backed Linux user accounts for shared/lab machines (school computer rooms, cyber cafes, kiosks). Every reboot wipes the user's home directory and restores a clean baseline — no persistent state, no manual cleanup, no accumulated junk/malware across sessions.

## Problem

Shared-use machines (driving school lab, public terminals) need each session to start from a known-clean state. Traditional approaches (manual cleanup scripts, re-imaging, Deep Freeze-style commercial tools) are slow, Windows-centric, or require extra software licenses.

## Solution

- User's home directory is mounted as `tmpfs` (RAM-backed, non-persistent).
- A `systemd` oneshot service repopulates the home from `/etc/skel` before the display manager starts.
- Reboot = instant reset. No disk writes to clean, no snapshot rollback needed.

## Architecture

Boot
└─ home-<user>.mount   (tmpfs mounted)
└─ student-home-init.service   (cp /etc/skel → home, chown)
└─ display-manager.service  (login screen)

## Repo structure

linux-ephemeral-lab/
├── ansible/
│   ├── inventory.ini
│   └── playbook.yml       # configures tmpfs home + systemd unit
└── proxmox/
└── template-notes.md  # golden VM → template → clone workflow


## Usage

### Ansible (bare metal or existing VM)

```bash
cd ansible
ansible-playbook -i inventory.ini playbook.yml
```

Configures:
- `student` user with tmpfs home (default 512M, `/home/student`)
- systemd unit to reset home from `/etc/skel` on every boot

### Proxmox (fleet deployment)

1. Build one VM, run the Ansible playbook against it.
2. Convert to template: `qm template <VMID>`
3. Clone per classroom PC: `qm clone <TEMPLATE_ID> <NEWID> --name student-pc-01 --full`

See `proxmox/template-notes.md` for full steps.

## Notes / limitations

- Resets on **reboot**, not on logout. Suits environments where users restart between sessions (driving school labs, exam rooms).
- tmpfs consumes RAM — size accordingly (`tmpfs_size` var in playbook).
- Not a substitute for full disk-level protection if persistence outside `/home` matters (browser cache elsewhere, `/tmp`, etc.) — extend the playbook if needed.

## License

MIT
