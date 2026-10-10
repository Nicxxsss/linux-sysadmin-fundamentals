# Debian server baseline

For the first lab, I created a clean Debian server that will be reused for the remaining Linux administration exercises.

The VM runs on my Ubuntu laptop using KVM/QEMU and virt-manager. I assigned 2 virtual CPUs, 2 GB of memory and a 20 GB QCOW2 disk. I kept the default NAT network because the VM only needs outbound Internet access and SSH access from the host.

## Minimal installation

I installed Debian without a graphical desktop. During software selection, I kept only the SSH server and standard system utilities.

The configured hostname is:

```text
debian-sysadmin-lab
```

After the first login, I updated the installed packages:

```bash
sudo apt update
sudo apt full-upgrade -y
```

## Checking the system

I checked the hostname, kernel, architecture, virtualization type and operating system release.

The VM is running Debian 13.7 (`trixie`) on `amd64`, and the operating system detects KVM as the virtualization environment.

The collected output is available in [`system-info.txt`](../../evidence/01-server-baseline/system-info.txt).

I also checked the assigned processors, memory and virtual disk using:

```bash
nproc
free -h
lsblk
```

Those results are stored in [`resources.txt`](../../evidence/01-server-baseline/resources.txt).

## Network and SSH validation

The VM received `192.168.122.177/24` through DHCP on `enp1s0`, with `192.168.122.1` as the default gateway.

I tested Internet connectivity with an IP address first:

```bash
ping -c 3 1.1.1.1
```

I then tested name resolution separately:

```bash
ping -c 3 debian.org
```

Both tests completed with no packet loss. Testing them separately helps distinguish a general connectivity problem from a DNS problem.

I also confirmed that the SSH service was active and checked for failed systemd units:

```bash
systemctl is-active ssh
systemctl --failed --no-pager
```

SSH returned `active`, and systemd reported no failed units. The complete output is in [`validation.txt`](../../evidence/01-server-baseline/validation.txt).

The IP address was assigned by DHCP, so it may change when the virtual network or VM is restarted.

## Baseline snapshot

After updating and validating the VM, I shut it down and created an internal virt-manager snapshot named:

```text
baseline-updated-install
```

This provides a clean recovery point before making changes in later labs.
