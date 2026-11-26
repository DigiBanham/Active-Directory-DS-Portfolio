# Network Services and Client Integration

## Overview

DC1 provides network services to the isolated VirtualBox internal network. DHCP supplies client IPv4 configuration, while Routing and Remote Access (RRAS) with Network Address Translation (NAT) allows the internal client to use DC1's external VirtualBox NAT connection.

## DHCP configuration

The Windows DHCP Server role was deployed and authorized in Active Directory. An IPv4 scope was configured for the internal `172.16.0.0/24` network.

| Scope setting | Configuration |
|---|---|
| Network | `172.16.0.0/24` |
| Subnet mask | `255.255.255.0` |
| Address pool | `172.16.0.100`–`172.16.0.200` |
| Router / default gateway | `172.16.0.1` |
| DNS server | `172.16.0.1` |

The scope provides CLIENT1 with an address on the internal subnet and directs it to DC1 for DNS and gateway services.

## Routing and NAT

RRAS was configured on DC1 to route traffic from the internal virtual network through the External adapter. NAT allows CLIENT1 to use DC1's outbound connection without placing the client directly on the external network.

The two interfaces have separate roles:

- **External:** VirtualBox NAT connection used by DC1 for outbound connectivity.
- **Internal:** Isolated network shared by DC1 and CLIENT1.

## Client domain integration

CLIENT1 was configured to use the internal network, obtain its IPv4 settings from DC1, and use the domain controller for DNS. The Windows 11 client was joined to `juancloudsolutions.com` and domain connectivity was tested.

The client integration work included checking the assigned network configuration, domain membership, domain authentication, DNS resolution, and network connectivity.

## Validation commands

The following commands are useful when inspecting client configuration and domain connectivity:

```powershell
# Inspect assigned addresses, DNS, and gateway
ipconfig /all

# Inspect domain membership
(Get-CimInstance Win32_ComputerSystem).Domain

# Query the domain controller's DNS record through DC1
Resolve-DnsName "dc1.juancloudsolutions.com" -Server 172.16.0.1

# Inspect the active domain logon server
$env:LOGONSERVER
```

These are diagnostic examples. The supplied archive does not include DHCP lease or RRAS console screenshots, so those should be added if you want this section to have the same visual evidence depth as the DNS and AD DS sections.

## Result

The lab connects a Windows 11 domain client to the Active Directory environment and provides it with DHCP-based network settings. DC1 supplies DNS and routed outbound access for the isolated internal network.
