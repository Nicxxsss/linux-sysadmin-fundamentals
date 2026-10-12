# Scheduled Health Check

This lab uses a Bash script and a systemd timer to collect a small set of system health indicators every 15 minutes.

The check reports:

- root filesystem usage;
- number of failed systemd units;
- SSH service state;
- one, five, and fifteen-minute load averages.

## Health-check script

The script reads the current system state and prints one structured line:

```text
health-check root_usage=7% failed_units=0 ssh=active load="0.00 0.00 0.00"
```

Filesystem usage is converted to a numeric value so it can be compared with a threshold. If root filesystem usage reaches `80%`, the script prints an additional warning.

Before installation, I validated the Bash syntax without executing the script:

```bash
bash -n /usr/local/bin/lab-health-check
```

The script is owned by root and has mode `0755`. Only root can modify it, while the service account can read and execute it.

## Service unit

The systemd service uses `Type=oneshot` because the script runs once, reports the results, and exits.

It runs as the existing `labsvc` service account instead of root:

```ini
User=labsvc
Group=labsvc
```

The unit also includes several restrictions:

```ini
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
```

These settings prevent privilege escalation, provide a private temporary directory, make the system filesystem read-only from the service, and block access to home directories.

## Timer

The timer uses the following schedule:

```ini
OnCalendar=*:0/15
Persistent=true
AccuracySec=1s
```

It runs at minutes `00`, `15`, `30`, and `45` of each hour. `Persistent=true` allows systemd to account for a scheduled run that was missed while the timer was inactive.

The timer was enabled with:

```bash
sudo systemctl enable --now lab-health-check.timer
```

I also started the service manually once to validate it without waiting for the next scheduled activation.

The journal confirmed that the script ran successfully, found no failed units, reported SSH as active, and exited with status `0`.

## Files

- [Health-check script](files/lab-health-check)
- [Service unit](files/lab-health-check.service)
- [Timer unit](files/lab-health-check.timer)

## Evidence

- [Timer and service validation](../../evidence/10-scheduled-health-check/health-check-validation.txt)
