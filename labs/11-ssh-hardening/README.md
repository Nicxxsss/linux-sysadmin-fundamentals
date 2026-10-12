# SSH Hardening

This lab replaces password-based SSH access with a dedicated Ed25519 key and reduces the authentication methods exposed by the Debian server.

Because an incorrect SSH configuration can cause a lockout, I created a VM snapshot and kept an existing session open until a second key-based connection succeeded.

## Key-based authentication

I generated a dedicated Ed25519 key on the client and installed only its public key in the `user1` account.

The private key remains outside the repository under `~/.ssh`. The public key is also excluded because the project only needs its fingerprint as evidence.

Before changing the server configuration, I forced a connection that could use only the selected key:

```bash
ssh \
  -i ~/.ssh/linux_sysadmin_lab \
  -o IdentitiesOnly=yes \
  -o PasswordAuthentication=no \
  user1@192.168.122.177
```

The connection succeeded, confirming that key authentication was ready before password access was disabled.

## Server configuration

The hardening settings were placed in a separate OpenSSH drop-in file:

```text
/etc/ssh/sshd_config.d/00-lab-hardening.conf
```

The configuration applies the following controls:

```text
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
MaxAuthTries 3
X11Forwarding no
AllowUsers user1
```

These settings disable direct root login, password and keyboard-interactive authentication, reduce the number of authentication attempts, disable X11 forwarding, and restrict SSH access to the administrative account used by this lab.

The file is owned by root with mode `0644`.

## Validation

Before reloading SSH, I checked the configuration syntax:

```bash
sudo sshd -t
```

The command returned exit status `0`. I also used `sshd -T` to inspect the effective configuration rather than assuming that the drop-in file had taken effect.

The SSH service was reloaded without terminating the existing session:

```bash
sudo systemctl reload ssh
```

From a second terminal, key-based authentication succeeded and returned the identity of `user1`.

I then disabled public-key authentication on the client and requested password authentication only. The attempt failed with:

```text
Permission denied (publickey).
```

This showed that the server no longer offered password authentication. The original session was closed only after the new key-based connection had been confirmed.

## Recovery

If the drop-in configuration causes a problem, it can be removed from the VM console:

```bash
sudo rm /etc/ssh/sshd_config.d/00-lab-hardening.conf
sudo sshd -t
sudo systemctl reload ssh
```

The snapshot created before the change provides an additional recovery option.

## Files

- [OpenSSH hardening configuration](files/00-lab-hardening.conf)

## Evidence

- [Configuration before hardening](../../evidence/11-ssh-hardening/ssh-before-hardening.txt)
- [Client authentication tests](../../evidence/11-ssh-hardening/ssh-client-tests.txt)
- [Effective configuration and server logs](../../evidence/11-ssh-hardening/ssh-server-validation.txt)
