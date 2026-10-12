# Backup and Restore

This lab creates and validates a backup of the custom `lab-heartbeat` service built earlier in the project.

The backup contains both files required to reconstruct the service:

- `/usr/local/bin/lab-heartbeat`
- `/etc/systemd/system/lab-heartbeat.service`

## Source files

Before creating the backup, I checked the ownership and permissions of both files:

```text
755 root:root /usr/local/bin/lab-heartbeat
644 root:root /etc/systemd/system/lab-heartbeat.service
```

The script must remain executable, while the systemd unit should only be writable by root. Preserving this metadata is part of a valid restoration.

## Creating the backup

I created a root-only backup directory:

```bash
sudo install -d -m 700 /var/backups/sysadmin-lab
```

The files were archived with paths relative to the root filesystem:

```bash
sudo tar \
  --create \
  --gzip \
  --file /var/backups/sysadmin-lab/lab-heartbeat.tar.gz \
  --directory / \
  usr/local/bin/lab-heartbeat \
  etc/systemd/system/lab-heartbeat.service
```

Using relative paths avoids writing directly to the live filesystem when the archive is extracted into another directory.

## Integrity check

I generated a SHA-256 checksum for the archive:

```bash
sudo sha256sum /var/backups/sysadmin-lab/lab-heartbeat.tar.gz
```

The verification returned `OK`, confirming that the archive still matched the recorded checksum.

A checksum can detect changes or corruption, but it does not prove that the files can be successfully restored.

## Restore test

I extracted the archive into a temporary directory instead of overwriting the live service files:

```bash
restore_dir=$(sudo mktemp -d /tmp/lab-restore.XXXXXX)

sudo tar \
  --extract \
  --gzip \
  --file /var/backups/sysadmin-lab/lab-heartbeat.tar.gz \
  --directory "$restore_dir"
```

The restored files were compared with the originals using `cmp`. Both comparisons matched.

I also checked their metadata after extraction:

```text
755 root:root lab-heartbeat
644 root:root lab-heartbeat.service
```

This confirmed that the backup preserved the file contents, ownership, and permissions needed to rebuild the service.

The temporary restoration directory was removed after validation. The original service files were not modified during the test.

## Evidence

- [Backup and integrity validation](../../evidence/09-backup-and-restore/backup-validation.txt)
- [Restore validation](../../evidence/09-backup-and-restore/restore-validation.txt)
