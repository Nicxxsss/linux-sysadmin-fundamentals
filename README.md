# Linux Sysadmin Fundamentals

A hands-on Linux administration project built on a Debian virtual machine.

I created this repository to practice routine administration work as a connected system rather than as a collection of isolated commands. Each lab starts with a small operational objective, records the commands and results, and keeps the raw evidence used to validate the outcome.

Several labs also include controlled failures. These include an invalid systemd executable, a failed SSH login, an occupied network port, a blocked connection, and a test restoration. The troubleshooting process is documented alongside the successful result.

## Environment

- Debian GNU/Linux 13 (trixie)
- KVM/libvirt virtual machine
- 2 virtual CPUs
- 2 GiB RAM
- 20 GiB virtual disk
- NAT-based virtual network
- systemd
- Bash
- OpenSSH
- nftables

The Ubuntu host was used as the administrative workstation. The Debian VM was managed remotely through SSH, except when console access or a recovery path was required.

## Labs

1. [Server baseline](labs/01-server-baseline/README.md)
   Recorded the operating system, kernel, architecture, virtual resources, interfaces, routing, DNS, SSH state, and failed units.

2. [Shared directory permissions](labs/02-shared-directory-permissions/README.md)
   Created users and a shared group, configured a setgid directory and default ACL, and tested authorized and unauthorized access.

3. [Package management](labs/03-package-management/README.md)
   Researched, installed, validated, and removed a Debian package while checking its repository source and APT history.

4. [Process management](labs/04-process-management/README.md)
   Inspected process relationships and states, generated controlled CPU activity, sent `SIGTERM`, and verified process termination.

5. [Custom systemd service](labs/05-systemd-service/README.md)
   Created a service account, script, and systemd unit, then introduced an invalid executable path to investigate and recover from `203/EXEC`.

6. [Log investigation](labs/06-log-investigation/README.md)
   Generated a failed SSH login and reconstructed the event through the systemd journal, including structured metadata and authentication messages.

7. [Network troubleshooting](labs/07-network-troubleshooting/README.md)
   Diagnosed an incorrect port, compared loopback and all-interface bindings, and resolved a real `Address already in use` conflict.

8. [Disk usage investigation](labs/08-disk-usage-investigation/README.md)
   Checked filesystems and inodes, created controlled disk usage, located large files with `du` and `find`, and validated cleanup.

9. [Backup and restore](labs/09-backup-and-restore/README.md)
   Archived the custom service, generated a SHA-256 checksum, restored it into a temporary location, and compared content and permissions.

10. [Scheduled health check](labs/10-scheduled-health-check/README.md)
    Built a Bash health-check script and scheduled it with a systemd timer running under a restricted service account.

11. [SSH hardening](labs/11-ssh-hardening/README.md)
    Replaced password authentication with an Ed25519 key, disabled direct root login, limited accepted users, and tested both successful and rejected authentication paths.

12. [Firewall management](labs/12-firewall-management/README.md)
    Created a persistent nftables ruleset with a default-drop input policy and verified blocked, temporarily allowed, and blocked-again access to a test service.

## Repository layout

```text
.
├── labs/
│   ├── 01-server-baseline/
│   ├── ...
│   └── 12-firewall-management/
├── evidence/
│   ├── 01-server-baseline/
│   ├── ...
│   ├── 12-firewall-management/
│   └── final-validation.txt
└── README.md
```

The `labs` directory contains the explanation for each exercise and any scripts or configuration files created during it. The `evidence` directory contains captured command output, service states, logs, test results, and failure messages.

Private SSH keys are not stored in the repository. The backup archive also remains on the VM; the repository contains only its checksum, archive listing, and restoration results.

## Final validation

After the final configuration, I rebooted the VM and confirmed that:

- key-based SSH access still worked;
- the SSH and nftables services were active and enabled;
- the nftables input rules were loaded from persistent configuration;
- the health-check timer remained active and scheduled;
- systemd reported no failed units.

The captured result is available in [final-validation.txt](evidence/final-validation.txt).

## Notes

This project was built in an isolated lab environment. Firewall, SSH, permissions, and service configurations should be reviewed and adapted before being used on another machine.

Snapshots and rollback paths were created before changes that could interrupt remote access.
