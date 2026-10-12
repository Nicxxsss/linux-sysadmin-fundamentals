# Firewall Management

This lab configures a stateful host firewall on the Debian server using nftables.

The initial inspection showed that the nftables package was installed, but its service was disabled and the active ruleset was empty. SSH was listening on port 22, while DHCP used UDP ports 68 and 546.

## Ruleset design

The firewall uses an `inet` table so the same rules can process IPv4 and IPv6 traffic.

The input chain follows a default-deny approach:

```text
type filter hook input priority 0; policy drop;
```

Only the traffic required by the server is accepted:

- loopback traffic;
- established and related connections;
- new SSH connections on TCP port 22;
- IPv4 and IPv6 ICMP;
- IPv4 and IPv6 DHCP replies.

Invalid connections are dropped. Forwarded traffic is also denied because this VM is not acting as a router, while outbound traffic remains allowed.

## Safe application

Before changing the firewall, I created a VM snapshot and validated the configuration without loading it:

```bash
sudo nft --check --file /etc/nftables.conf
```

I also scheduled a temporary systemd rollback:

```bash
sudo systemd-run \
  --unit=nftables-lab-rollback \
  --on-active=5m \
  /usr/sbin/nft flush ruleset
```

If the new rules had blocked SSH, the ruleset would have been cleared automatically after five minutes. After confirming that a new SSH connection worked, I cancelled the rollback.

The rules were stored in `/etc/nftables.conf`, and the nftables service was enabled so they are loaded during boot.

## Port validation

I started a temporary Python HTTP server listening on `0.0.0.0:8080`.

Although `ss` confirmed that the process was listening, a request from the host timed out because the firewall did not allow new connections to port 8080:

```text
Exit status: 28
```

I inserted a temporary allow rule for TCP port 8080. The same request then succeeded and returned:

```text
firewall validation
Exit status: 0
```

After deleting the temporary rule by its nftables handle, the request timed out again. The HTTP server remained active during these tests, isolating the firewall rule as the reason for the change in reachability.

A new SSH connection continued to succeed through the permanent TCP port 22 rule.

## Final state

The temporary HTTP server and its allow rule were removed. The remaining listening network service was SSH.

The persistent ruleset passed syntax validation, the nftables service was enabled and active, DNS resolution continued to work, and the VM retained outbound network connectivity.

## Files

- [Persistent nftables configuration](files/nftables.conf)

## Evidence

- [Client connectivity tests](../../evidence/12-firewall-management/firewall-client-tests.txt)
- [Ruleset and server validation](../../evidence/12-firewall-management/firewall-server-validation.txt)

## References

- [Debian nftables documentation](https://wiki.debian.org/nftables)
- [nftables server ruleset example](https://wiki.nftables.org/wiki-nftables/index.php/Simple_ruleset_for_a_server)
