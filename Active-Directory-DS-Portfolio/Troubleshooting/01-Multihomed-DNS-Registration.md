# Troubleshooting: DNS Registration on a Multihomed Domain Controller

## Problem

DC1 has two network interfaces: an External adapter connected to VirtualBox NAT and an Internal adapter connected to the isolated domain network. Initial DNS queries showed that addresses from both adapters had been registered in the Active Directory DNS zone. The External NAT address and IPv6 addresses were not appropriate host records for internal domain clients.

## Initial observations

| Adapter | Address observed | Intended role |
|---|---|---|
| External | `10.0.2.15` (DHCP) | Outbound connectivity through VirtualBox NAT; should not register in the internal zone |
| External | IPv6 ULA and link-local address | Not intended for internal domain-client host records |
| Internal | `172.16.0.1/24` | Stable address for AD DS and DNS |
| Internal | IPv6 link-local address | Not used for this IPv4-focused lab design |

The initial DNS zone contained an A record for the External address and several AAAA records. The External adapter was also configured to register its connection address.

## Investigation and corrective actions

1. Inspected the IPv4 and IPv6 addresses assigned to both adapters.
2. Disabled DNS registration on the External adapter.
3. Restricted the DNS Server service to listen only on `172.16.0.1`.
4. Set the IPv4 DNS client address on both adapters to `172.16.0.1`.
5. Removed the unwanted External A record and the unwanted AAAA records from the DC1 host record.
6. Queried the authoritative zone and tested direct DNS resolution against DC1.

The External adapter registration setting was changed with:

```powershell
Set-DnsClient `
    -InterfaceAlias "External" `
    -RegisterThisConnectionsAddress $false
```

The final DNS Server listening interface was restricted through DNS Manager to the Internal address `172.16.0.1`.

## Verification

The authoritative zone contained only the intended DC1 A record (`172.16.0.1`) and no DC1 AAAA records. Direct DNS queries returned the intended internal address:

```powershell
Resolve-DnsName `
    dc1.juancloudsolutions.com `
    -Type A `
    -Server 172.16.0.1 `
    -DnsOnly `
    -NoHostsFile

nslookup dc1.juancloudsolutions.com 172.16.0.1
```

Both direct tests returned `172.16.0.1`.

## Diagnostic caveat

After the authoritative records had been corrected and the local DNS Client cache cleared, some standard `Resolve-DnsName` invocations continued to display the old External address in their output. The behavior was not conclusively explained. Direct DNS-only resolution against `172.16.0.1` and `nslookup` returned only the intended address, so no further DNS-zone changes were made based on the unexplained output.

## Outcome

The External adapter no longer registers its address in the internal domain zone, DNS listens on the Internal IPv4 interface, and the authoritative DC1 host record contains the intended internal address. The unresolved standard-query behavior is retained as a diagnostic note rather than being presented as a fully explained issue.

## Evidence

- [Initial SRV and host-record query](../Screenshots/Desktop-Experience/Active-Directory/10-DNS-SRV-Record-Initial-Verification.png)
- [Adapter address inspection](../Screenshots/Desktop-Experience/Active-Directory/11-Associating-IP-Addresses-To-NAdapter.png)
- [DNS listening interfaces](../Screenshots/Desktop-Experience/Active-Directory/12-DNS-Listening-Interfaces.png)
- [DNS output discrepancy](../Screenshots/Desktop-Experience/Active-Directory/13-DNS-Output-Discrepancy.png)
- [Final IPv4 DNS client configuration](../Screenshots/Desktop-Experience/Active-Directory/24-DNS-Client-Configuration-Verification.png)
