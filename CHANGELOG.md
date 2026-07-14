# Changelog

## [Unreleased]
- Initial ansible playbook: tmpfs home + systemd reset unit
- Proxmox template/clone workflow notes

## [0.1.0] - 2026-07-07
- Repo created

## [0.2.0] - 2026-07-14
- Converted playbook to role (student_tmpfs)
- Added USB mass-storage block (udev)
- Derived username from inventory hostname
- Validated end-to-end on 2 AlmaLinux test VMs: tmpfs reset confirmed working across reboot
