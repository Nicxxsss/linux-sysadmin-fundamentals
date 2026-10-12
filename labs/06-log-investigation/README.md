# Investigating a failed SSH login

In this lab, I generated a failed SSH login against the Debian VM and used the systemd journal to reconstruct what happened.

The test was performed from the Ubuntu host using an account that did not exist:

```bash
ssh \
  -o PubkeyAuthentication=no \
  -o PreferredAuthentications=password \
  -o NumberOfPasswordPrompts=1 \
  invaliduser@192.168.122.177
```

I entered a false test password and the server rejected the login.

## Finding the event

My first query used the process name:

```bash
sudo journalctl _COMM=sshd --since "5 minutes ago"
```

It returned no entries. The event had been recorded, but the process field used by this Debian version was `sshd-session`, not `sshd`.

I changed the search to the systemd unit:

```bash
sudo journalctl -u ssh.service --since "15 minutes ago"
```

This returned both the failed attempt and the valid login I performed immediately afterward.

The timeline showed:

- An invalid username from `192.168.122.1`;
- PAM checking an unknown user;
- An authentication failure;
- A failed password event;
- The connection closing during `preauth`;
- A successful login for `user1`;
- The valid user session opening.

`192.168.122.1` was the Ubuntu host on the KVM NAT network.

The filtered timeline is stored in [`ssh-auth-timeline.txt`](../../evidence/06-log-investigation/ssh-auth-timeline.txt).

## Inspecting structured journal fields

The failed connection used PID `5683`. I inspected the complete journal fields with:

```bash
sudo journalctl _PID=5683 -o verbose
```

The fields showed:

```text
_COMM=sshd-session
_SYSTEMD_UNIT=ssh.service
_EXE=/usr/lib/openssh/sshd-session
_CMDLINE="sshd-session: unknown [priv]"
MESSAGE=Invalid user invaliduser from 192.168.122.1
```

This explained why the original `_COMM=sshd` filter did not match.

The entries also used priorities `5` and `6`, which correspond to notice and informational messages. A search restricted to warnings and more severe events could therefore miss this failed authentication. Log severity alone is not enough to decide whether an event is security-relevant.

The structured output is available in [`ssh-structured-event.txt`](../../evidence/06-log-investigation/ssh-structured-event.txt).

The `_UID=0` field represented the privileged SSH process recording the event. It did not mean that the failed user obtained root access.

## Filtering another service

I also queried the heartbeat service created in the previous lab:

```bash
sudo journalctl -u lab-heartbeat.service -n 5 --no-pager
```

The `-u` option selected one unit, while `-n 5` limited the result to its five most recent entries.

The output is stored in [`heartbeat-filter.txt`](../../evidence/06-log-investigation/heartbeat-filter.txt).

After collecting the logs, I stopped and disabled the heartbeat service so it would not continue generating entries:

```bash
sudo systemctl disable --now lab-heartbeat.service
```

Its final state is recorded in [`heartbeat-final-state.txt`](../../evidence/06-log-investigation/heartbeat-final-state.txt).
