# Troubleshooting: DNS Forwarder Reachability

## Problem

The DNS Server on DC1 was configured with two forwarders. DNS diagnostics and direct queries indicated that the IPv6 forwarder could not be reached, while the IPv4 forwarder was responsive. An unreachable forwarder can delay external name resolution and should not be retained as a configured upstream server without a working path.

## Direct reachability tests

Each forwarder was tested using `Resolve-DnsName` over both UDP (the default) and TCP (`-TcpOnly`).

| Forwarder | Transport | Result |
|---|---|---|
| `192.168.1.1` | UDP | Successful external hostname resolution |
| `192.168.1.1` | TCP | Successful external hostname resolution |
| `fd17:625c:f037:2::3` | UDP | Query timed out |
| `fd17:625c:f037:2::3` | TCP | Socket communication error |

The IPv6 address was a ULA from the local network's DNS service but was not reachable from DC1 during testing.

## Corrective action

Microsoft's [DNS troubleshooting guidance](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/troubleshoot-dns-guidance) recommends verifying forwarder reachability and removing forwarders that are unreachable. Based on the direct UDP and TCP tests, the unreachable IPv6 forwarder was removed:

```powershell
Remove-DnsServerForwarder `
    -IPAddress "fd17:625c:f037:2::3" `
    -Force `
    -PassThru
```

The remaining configuration was inspected with:

```powershell
Get-DnsServerForwarder |
    Format-List IPAddress,UseRootHint,Timeout,EnableReordering
```

| Setting | Final value |
|---|---|
| Forwarder | `192.168.1.1` |
| Use root hints | `True` |
| Timeout | 3 seconds |
| Enable reordering | `True` |

## Verification

After clearing the DNS Server cache, an external hostname was queried directly through DC1's internal DNS address:

```powershell
Clear-DnsServerCache -Force

Resolve-DnsName `
    -Name "www.microsoft.com" `
    -Server "172.16.0.1" `
    -DnsOnly
```

The query returned a CNAME chain and IPv4/IPv6 address records for the destination. This demonstrates that DC1 could resolve an external hostname after the forwarder cleanup. It does not, by itself, identify whether the response was obtained from the configured IPv4 forwarder or through root-hint recursion.

The subsequent `dcdiag /test:dns /v` results showed that the internal DNS server and the remaining IPv4 forwarder passed their DNS tests. The overall domain DNS test passed, with a `Basic` warning for the External adapter's DHCP-assigned VirtualBox NAT address. That dynamic address is intentional in this lab design.

## Outcome

The unreachable IPv6 forwarder was removed, the reachable IPv4 forwarder was retained, and external hostname resolution through DC1 was verified.

## Evidence

- [IPv4 forwarder tests — command](../Screenshots/Desktop-Experience/Active-Directory/25a-IPv4-Forwarder-Reachability-Tests.png)
- [IPv4 forwarder tests — results](../Screenshots/Desktop-Experience/Active-Directory/25b-IPv4-Forwarder-Reachability-Tests.png)
- [IPv6 forwarder tests — commands](../Screenshots/Desktop-Experience/Active-Directory/25c-IPv6-Forwarder-Reachability-Failures.png)
- [IPv6 forwarder tests — failures](../Screenshots/Desktop-Experience/Active-Directory/25d-IPv6-Forwarder-Reachability-Failures.png)
- [Removal of unreachable forwarder](../Screenshots/Desktop-Experience/Active-Directory/26-Removing-Unreachable-DNS-Forwarder.png)
- [Final forwarder configuration](../Screenshots/Desktop-Experience/Active-Directory/27-Final-DNS-Forwarder-Configuration.png)
- [External hostname resolution](../Screenshots/Desktop-Experience/Active-Directory/28-Exernal-DNS-Resolution-Verification.png)
- [Final DNS diagnostics](../Screenshots/Desktop-Experience/Active-Directory/29a-Final-DCDIAG-DNS-Results.png) and [summary](../Screenshots/Desktop-Experience/Active-Directory/29b-Final-DCDIAG-DNS-Results.png)
