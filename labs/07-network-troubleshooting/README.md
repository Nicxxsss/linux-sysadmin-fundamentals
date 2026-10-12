# Network Troubleshooting

This lab investigates a small HTTP service that is unavailable through the expected address or port. The goal is to use the system's network state to locate the problem instead of changing settings at random.

## Environment

The tests were performed on a Debian virtual machine connected through a KVM NAT network. The VM received `192.168.122.177/24`, with `192.168.122.1` as its default gateway and DNS server.

Before starting the HTTP service, I checked the interface addresses, routing table, DNS resolution, and listening sockets:

```bash
ip -brief address
ip route
cat /etc/resolv.conf
getent ahosts debian.org
sudo ss -lntup
```

The interface was up, a default route existed, DNS resolution worked, and SSH was listening on port 22.

## Testing the wrong port

I started a temporary Python HTTP server on port `8080` and intentionally attempted to connect to port `8000`:

```bash
curl --max-time 3 http://127.0.0.1:8000/
```

The connection failed with curl exit status `7`. Checking the sockets with `ss` showed that the application was listening on `8080`, not `8000`.

This confirmed that the failure was caused by using the wrong destination port.

## Testing the bind address

The server was first bound only to the loopback address:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
```

Requests to `127.0.0.1:8080` succeeded, but requests to `192.168.122.177:8080` failed. A listener bound to `127.0.0.1` is available only through the local loopback interface.

I then restarted the service on all local IPv4 interfaces:

```bash
python3 -m http.server 8080 --bind 0.0.0.0
```

`ss` reported the listener as `0.0.0.0:8080`, and the service became reachable through the VM interface address.

`0.0.0.0` is a listening address, not an address that clients use to connect. Clients use one of the machine's actual IP addresses, while routing and firewall rules still determine whether remote access is possible.

## Port conflict

While repeating the test, an older Python process was still listening on port `8080`. A second server could not claim the same address and port and exited with:

```text
OSError: [Errno 98] Address already in use
```

I identified the process holding the port with:

```bash
ss -lntp | grep ':8080'
```

After terminating that process with `SIGTERM`, I repeated the tests successfully and confirmed that no listener remained on port `8080`.

This demonstrated why the current socket state and owning process should be checked before assuming that an application started correctly.

## Evidence

- [Network baseline](../../evidence/07-network-troubleshooting/network-baseline.txt)
- [Port conflict](../../evidence/07-network-troubleshooting/port-conflict.txt)
- [Port and bind-address tests](../../evidence/07-network-troubleshooting/port-troubleshooting.txt)
