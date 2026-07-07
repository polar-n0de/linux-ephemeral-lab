# Proxmox Template Notes

## 1. Build the base template

Use [proxmox-template-creator](https://github.com/polar-n0de/proxmox-template-creator):

```bash
git clone https://github.com/polar-n0de/proxmox-template-creator.git
cd proxmox-template-creator
sudo ./create_template.sh
```

Choose: Standard or Minimal profile, Ansible packages = yes.

## 2. Clone from template, run playbook

```bash
qm clone <TEMPLATE_ID> <NEWID> --name student-pc-01 --full
qm set <NEWID> --ipconfig0 ip=dhcp
qm start <NEWID>
```

Then configure the clone with this repo's Ansible playbook:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbook.yml
```

## 3. (Optional) Bake config into a new template

Once configured and verified (reboot, check tmpfs home + USB block work), convert the clone itself back into a template for future fast deployment:

```bash
qm template <NEWID>
```

## Notes
- Hostname per clone drives `student_user` var (student-pc-01 → student_01) — set via `qm set <NEWID> --name` or cloud-init.
- tmpfs/systemd/udev config lives in this repo's Ansible role, not in proxmox-template-creator — the two repos are complementary (base image vs. lab-specific hardening).
