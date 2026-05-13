# grub-backup

`grub-backup` is a Bash utility for creating, restoring, listing, and deleting GRUB configuration backups, similar to `git stash`.

The project is intended to safely save the current bootloader state before configuration changes, upgrades, or manual edits.

---

## Features

- Create backups of the current GRUB configuration
- Restore saved configurations
- List available backups
- Delete backups by name or delete all backups
- Named and unnamed backups
- Custom backup storage directories
- Automatic `update-grub` run after restore
- Full testing through a separate test script

---

## Backed Up Files

```text
/etc/default/grub
/etc/grub.d/
/boot/grub/grub.cfg
```

---

## Requirements

* Ubuntu / Xubuntu / Debian-based Linux
* Bash 4+
* `tar`
* `update-grub`
* Root privileges (`sudo`) for `save` and `restore`

---

## Installation

### Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/grub-backup.git
cd grub-backup
```

### Install system-wide

```bash
sudo install -m 755 grub-backup /usr/local/bin/grub-backup
sudo install -m 755 test-grub-backup /usr/local/bin/test-grub-backup
```

---

## Usage

### Create a backup

```bash
sudo grub-backup save
```

### Create a named backup

```bash
sudo grub-backup save -m before-kernel-update
```

### Use a custom storage directory

```bash
sudo grub-backup save -p ~/grub-backups -m custom-backup
```

---

### List backups

```bash
grub-backup list
```

```bash
grub-backup list -p ~/grub-backups
```

---

### Restore a backup

Without `-m`, `restore` restores the latest unnamed backup.

```bash
sudo grub-backup restore
```

With `-m`, `restore` restores the latest backup with that name.

```bash
sudo grub-backup restore -m before-kernel-update
```

```bash
sudo grub-backup restore -p ~/grub-backups -m custom-backup
```

---

### Delete backups with a specific name

This deletes all backups with the matching name.

```bash
grub-backup clear -m before-kernel-update
```

---

### Delete all backups

```bash
grub-backup clear --all
```

---

## Testing

### Quick check

```bash
./test-grub-backup ./grub-backup
```

### Full testing, including restore

```bash
sudo ./test-grub-backup ./grub-backup --with-restore
```

---

## Safety

> **Important:**
> The `restore` command modifies real GRUB system files.
> Use a virtual machine for safe restore testing.

---

## Example Workflow

```bash
sudo grub-backup save -m before-edit
sudo nano /etc/default/grub
sudo update-grub

# If something goes wrong:
sudo grub-backup restore -m before-edit
```

---

## License

MIT License

Free use, modification, and distribution are permitted.

---

## Why This Project Is Useful

GRUB changes can cause:

* a system that cannot boot;
* bootloader menu errors;
* kernel conflicts;
* loss of custom parameters.

`grub-backup` lets you quickly roll back to a working state.

---

## Contributing

Pull requests, issues, and suggestions are welcome.
