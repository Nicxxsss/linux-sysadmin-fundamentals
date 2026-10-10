# Creating and troubleshooting a systemd service

In this lab, I created a small background service, configured systemd to manage it and then introduced an incorrect executable path to practice diagnosing a failed unit.

## Service account

I created a dedicated system account instead of running the service as `root`:

```bash
sudo useradd \
  --system \
  --user-group \
  --no-create-home \
  --shell /usr/sbin/nologin \
  labsvc
```

The account has no home directory and cannot be used for an interactive login. Its purpose is limited to running the service.

## Heartbeat program

The service runs a small Bash loop that prints a timestamp every 30 seconds. The script used in the VM is available at [`files/lab-heartbeat`](files/lab-heartbeat).

Before involving systemd, I tested it directly as `labsvc`:

```bash
sudo -u labsvc timeout 3s /usr/local/bin/lab-heartbeat
```

The script printed a heartbeat and `timeout` returned status `124`, confirming that the program ran and was stopped only because the test reached its time limit.

Testing the program separately made it possible to distinguish script problems from systemd configuration problems.

## Unit configuration

I created `lab-heartbeat.service` under `/etc/systemd/system`. A copy is available at [`files/lab-heartbeat.service`](files/lab-heartbeat.service).

The unit runs as `labsvc` and uses:

```ini
Restart=on-failure
RestartSec=5s
```

This tells systemd to retry the service after an unexpected failure, waiting five seconds between attempts.

I validated and loaded the unit with:

```bash
sudo systemd-analyze verify /etc/systemd/system/lab-heartbeat.service
sudo systemctl daemon-reload
sudo systemctl enable --now lab-heartbeat.service
```

The service returned `active` and `enabled`.

The first journal query made as `user1` returned no entries because the account was not authorized to read messages produced by other users and system services. Running the query through `sudo` displayed the startup event and heartbeat messages. I kept journal access restricted instead of adding the regular account to a privileged logging group only for this test.

The working unit configuration and current state are recorded in [`service-state.txt`](../../evidence/05-systemd-service/service-state.txt).

## Controlled failure

I backed up the valid unit and changed its `ExecStart` value to a nonexistent path:

```ini
ExecStart=/usr/local/bin/lab-heartbeat-missing
```

After reloading systemd and restarting the service, the journal reported:

```text
Unable to locate executable
No such file or directory
status=203/EXEC
```

Status `203/EXEC` showed that systemd loaded the unit but could not execute the configured program.

Because the unit used `Restart=on-failure`, systemd tried again every five seconds. I stopped the service after collecting the diagnostic output to avoid leaving it in a restart loop.

## Recovery

I restored the known-good unit, validated it and reloaded systemd:

```bash
sudo systemd-analyze verify /etc/systemd/system/lab-heartbeat.service
sudo systemctl daemon-reload
sudo systemctl reset-failed lab-heartbeat.service
sudo systemctl start lab-heartbeat.service
```

The next journal entries showed the service starting successfully and producing heartbeat messages again.

The failure and recovery timeline is available in [`failure-recovery.txt`](../../evidence/05-systemd-service/failure-recovery.txt).
