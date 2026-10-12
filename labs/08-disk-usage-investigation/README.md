# Disk Usage Investigation

This lab simulates an unexpected increase in disk usage and follows a simple investigation process to identify and remove the responsible files.

## Storage baseline

I started by checking the available block devices, mounted filesystems, disk usage, and inode consumption:

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
df -hT
df -i
journalctl --disk-usage
```

The root filesystem used an `ext4` partition mounted from `/dev/vda1`. Both disk space and inode availability were checked because either resource can prevent new files from being created.

## Controlled disk usage

I created a temporary directory under `/var/tmp` containing two allocated files:

```bash
mkdir -p /var/tmp/disk-lab/cache

fallocate -l 64M /var/tmp/disk-lab/application.log
fallocate -l 24M /var/tmp/disk-lab/cache/archive.cache
```

The files represented an application log and a cached archive consuming more space than expected.

## Investigation

I inspected the directory recursively and sorted the results by size:

```bash
du -ah /var/tmp/disk-lab | sort -h
```

The directory occupied approximately `89 MiB`. The largest item was `application.log` at `64 MiB`, followed by `archive.cache` at `24 MiB`.

I then searched directly for files larger than `20 MiB`:

```bash
find /var/tmp/disk-lab -type f -size +20M \
  -printf '%s bytes %p\n' | sort -nr
```

This identified both files and displayed their exact sizes in bytes. Using `du` provided an overview of directory consumption, while `find` applied a specific threshold to locate unusually large files.

## Cleanup and validation

After identifying the controlled files, I removed them and their empty directories. I then confirmed that `/var/tmp/disk-lab` no longer existed.

A final filesystem check showed the root partition at approximately `7%` usage:

```bash
df -hT /var/tmp
```

Only files created specifically for this lab were removed.

## Evidence

- [Storage baseline](../../evidence/08-disk-usage-investigation/storage-baseline.txt)
- [Disk usage investigation](../../evidence/08-disk-usage-investigation/storage-investigation.txt)
- [Cleanup validation](../../evidence/08-disk-usage-investigation/storage-cleanup.txt)
