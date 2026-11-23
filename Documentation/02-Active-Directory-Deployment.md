# Active Directory Domain Services Deployment

## Overview

Active Directory Domain Services (AD DS) was deployed on `DC1`, a Windows Server 2025 Datacenter Evaluation virtual machine. The deployment established a new forest and domain for centralized identity, authentication, and directory management.

## Domain configuration

| Setting | Value |
|---|---|
| Domain controller | `DC1` |
| Domain FQDN | `juancloudsolutions.com` |
| NetBIOS domain | `JUANCLOUDSOLUTI` |
| Forest functional level | Windows Server 2025 |
| Domain functional level | Windows Server 2025 |
| Global Catalog | Enabled |
| DNS Server | Installed with the domain controller deployment |
| Domain controllers | One (`DC1`) |

## Deployment approach

The AD DS role was installed through Server Manager. DC1 was promoted through the Active Directory Domain Services Configuration Wizard using **Add a new forest** and the domain name `juancloudsolutions.com`.

The wizard was configured to install DNS, enable the Global Catalog, and use the Windows Server 2025 forest and domain functional levels. The Read Only Domain Controller option was not selected. A separate Directory Services Restore Mode (DSRM) password was configured and stored securely.

The wizard generated a PowerShell script that was reviewed as a reference for understanding the deployment. The forest creation and domain controller promotion were completed through the graphical wizard; the generated script was not executed.

## Deployment evidence

- [AD DS role selection](../Screenshots/Desktop-Experience/Active-Directory/01-ADDS-Install1.png)
- [New forest configuration](../Screenshots/Desktop-Experience/Active-Directory/03-New-Forest-Configuration.png)
- [Domain controller options](../Screenshots/Desktop-Experience/Active-Directory/04-Domain-Controller-Options.png)
- [Generated PowerShell script preview](../Screenshots/Desktop-Experience/Active-Directory/06-PS-Script-Install-ADDS.png)
- [Promotion completed](../Screenshots/Desktop-Experience/Active-Directory/07-ADDS-Promotion-Completed.png)

## Verification

The deployment was checked using Active Directory PowerShell cmdlets and DNS queries.

```powershell
# Verify the forest
Get-ADForest |
    Format-List Name,RootDomain,ForestMode,Domains

# Verify the domain
Get-ADDomain |
    Format-List DNSRoot,NetBIOSName,DomainMode

# Verify DC1
Get-ADDomainController -Identity DC1 |
    Format-List HostName,Domain,Forest,IPv4Address,IsGlobalCatalog,OperatingSystem

# Verify the AD-integrated DNS zone
Get-DnsServerZone -Name "juancloudsolutions.com" |
    Format-List ZoneName,ZoneType,IsDsIntegrated

# Query the LDAP service location record
Resolve-DnsName -Type SRV `
    "_ldap._tcp.dc._msdcs.juancloudsolutions.com" `
    -Server 172.16.0.1
```

The LDAP SRV query returned the domain controller target `dc1.juancloudsolutions.com` on TCP port 389. DNS configuration and the multihomed-server registration issue are covered in the [DNS troubleshooting case studies](../Troubleshooting/01-Multihomed-DNS-Registration.md) and [forwarder case study](../Troubleshooting/02-DNS-Forwarder-Reachability.md).

## Domain controller health checks

The following checks were used to confirm core service availability:

```powershell
# Run domain controller diagnostics
dcdiag /v

# Check important services
Get-Service -Name NTDS,DNS,Netlogon,DFSR

# Confirm SYSVOL and NETLOGON shares
net share
```

The supplied evidence shows the NTDS, DNS, Netlogon, and DFS Replication services running and the SYSVOL and NETLOGON shares present.

- [Forest and domain verification](../Screenshots/Desktop-Experience/Active-Directory/08-Forest-and-Domain-Verification.png)
- [Domain controller verification](../Screenshots/Desktop-Experience/Active-Directory/09-Domain-Controller-Verification.png)
- [Service status](../Screenshots/Desktop-Experience/Active-Directory/30a-Important-Services-for-ADDS.png)
- [SYSVOL and NETLOGON shares](../Screenshots/Desktop-Experience/Active-Directory/30b-Important-Shares-for-ADDS.png)

## Result

The new forest and domain were established, DC1 was promoted successfully, DNS was installed, and core directory services were verified. The environment provides the identity foundation for the DHCP, client integration, and account-administration exercises documented elsewhere in this portfolio.
