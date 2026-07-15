
# linux-ephemeral-lab

Ephemeral, tmpfs-backed Linux user accounts for shared/lab machines (school computer rooms, driving-test labs, kiosks). Every reboot wipes the user's home directory and restores a clean baseline. USB mass storage is blocked to prevent data exfiltration/cheating. No persistent state, no manual cleanup.

## Problem

Shared-use machines need each session to start from a known-clean state, without commercial Deep Freeze-style tools or full re-imaging.

## Solution

- User's home directory is mounted as `tmpfs` (RAM-backed, non-persistent).
- A `systemd` oneshot service repopulates the home from `/etc/skel` before the display manager starts.
- USB mass-storage devices are blocked via `udev` (keyboard/mouse unaffected).
- Reboot = instant reset. No disk cleanup, no snapshot rollback needed.

## End-to-end flow

proxmox-template-creator  →  qm clone  →  ansible (student_tmpfs role)  →  verified clean VM
(base OS template)         (per-PC VM)   (tmpfs + udev + systemd)      (optional: qm template)

1. Build a base OS template with [proxmox-template-creator](https://github.com/polar-n0de/proxmox-template-creator)
2. Clone it once per classroom PC
3. Run this repo's Ansible role against each clone
4. (Optional) Convert the configured clone back into a template for fast future redeploys

## Repo structure

```
linux-ephemeral-lab/
├── ansible/
│   ├── inventory.ini
│   └── playbook.yml
├── roles/
│   └── student_tmpfs/
│       ├── README.md
│       ├── defaults/main.yml
│       ├── tasks/main.yml
│       └── files/99-usb-block.rules
└── proxmox/
    └── template-notes.md
```
## Prerequisites

- Ansible >= 2.14
- Collection: `ansible.posix` (`ansible-galaxy collection install ansible.posix`)
- Target OS: tested on AlmaLinux 9 (should work on any systemd-based distro: RHEL/Rocky/Debian/Ubuntu)
- SSH key-based access to target hosts with sudo

## Usage

```bash
git clone https://github.com/polar-n0de/linux-ephemeral-lab.git
cd linux-ephemeral-lab
ansible-galaxy collection install ansible.posix
```

Edit `ansible/inventory.ini` with your target hostnames/IPs (hostname format `student-pc-NN` — username is auto-derived, e.g. `student-pc-01` → `student_01`).

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbook.yml
```

## Tested

Validated on 2x AlmaLinux 9 VMs (Proxmox):
- tmpfs home mounts correctly, mode 0700, per-user isolation
- systemd unit fires automatically on every boot (confirmed via `journalctl`/`systemctl status` timestamps matching boot time)
- File written to home before reboot is confirmed gone after reboot
- USB mass-storage udev rule deployed and reloaded successfully

## Limitations

- Resets on **reboot**, not on logout — suits environments where users restart between sessions.
- tmpfs consumes RAM — size via `tmpfs_size` role variable.
- Home-directory scope only; browser cache/`/tmp` persistence not addressed (extend the role if needed).

## Roadmap

- [ ] Idle auto-reboot timer
- [ ] Read-only root filesystem overlay
- [ ] Molecule test scaffolding

## License

MIT
