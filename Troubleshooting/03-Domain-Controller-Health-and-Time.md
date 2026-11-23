# Troubleshooting: Domain Controller Health and Windows Time

## Initial health assessment

A verbose domain controller diagnostic was run after the initial AD DS and DNS deployment:

```powershell
dcdiag /v
```

The initial SystemLog review identified a Windows Time event (Event ID 36). At the time, DC1 reported the Local CMOS Clock as its time source and had not successfully synchronized with an external time provider.

DC1 is the only domain controller in the forest and holds all five FSMO roles, including the forest-root PDC Emulator. In a single-domain-controller lab, the PDC Emulator has no upstream domain controller from which to obtain time, so it must be configured with an external reference.

## Why time synchronization matters

Kerberos authentication uses timestamps, so domain computers need reasonably synchronized clocks. Significant clock skew can cause authentication failures. The Windows Time service also follows the Active Directory time hierarchy, with the forest-root PDC Emulator at the top of the domain hierarchy.

## NTP connectivity test

Two NIST Internet Time Service hostnames were selected:

- `time-a-g.nist.gov`
- `time-d-b.nist.gov`

The hostnames resolved successfully. Connectivity to the primary peer was tested using:

```powershell
w32tm /stripchart `
    /computer:time-a-g.nist.gov `
    /samples:5 `
    /dataonly
```

The test returned time samples, confirming that DC1 could communicate with the selected NTP server.

## Configuration

DC1 was configured to use the two manual NTP peers and advertise itself as a reliable time source for domain clients:

```powershell
w32tm /config `
    /manualpeerlist:"time-a-g.nist.gov,0x8 time-d-b.nist.gov,0x8" `
    /syncfromflags:manual `
    /reliable:yes `
    /update

Restart-Service -Name W32Time
w32tm /resync /rediscover
```

The `0x8` peer flag configures the Windows NTP client to use client mode with the specified peers.

## Verification

The following commands were used to inspect the source, status, configuration, and peer state:

```powershell
w32tm /query /source
w32tm /query /status
w32tm /query /configuration
w32tm /query /peers
```

The verification showed:

| Setting | Result |
|---|---|
| Active source | `time-a-g.nist.gov,0x8` |
| Configured peers | `time-a-g.nist.gov` and `time-d-b.nist.gov` |
| Client type | `NTP` |
| Stratum | 2 |
| Leap indicator | 0 (no warning) |
| Subsequent DCDIAG Windows Time check | Passed |

The successful synchronization replaced the previous Local CMOS Clock source. Domain members can now obtain time through the Active Directory domain hierarchy, with DC1 at the top of this single-DC lab's hierarchy.

## Supporting domain controller checks

The supplied evidence also includes:

- `Get-Service -Name NTDS,DNS,Netlogon,DFSR`, showing the key services running.
- `net share`, showing the `SYSVOL` and `NETLOGON` shares.
- A subsequent DCDIAG result showing the Windows Time test passing.

## Outcome

DC1 was configured to synchronize with external NIST NTP peers, and the active source, Stratum, peer list, and subsequent DCDIAG result were verified. This corrects the initial time synchronization finding and establishes DC1 as the lab's reliable domain time source.

## Evidence

- [Initial DCDIAG findings](../Screenshots/Desktop-Experience/Active-Directory/19-dcdiag-errors.png)
- [Initial Windows Time status](../Screenshots/Desktop-Experience/Active-Directory/20-Initial-Windows-Time-Configuration.png)
- [Initial peer state](../Screenshots/Desktop-Experience/Active-Directory/20-Initial-Windows-Time-Configuration2.png)
- [External NTP connectivity test](../Screenshots/Desktop-Experience/Active-Directory/21-External-NTP-Connectivity-Test.png)
- [Windows Time status and configuration](../Screenshots/Desktop-Experience/Active-Directory/22-Windows-Time-Synchronization-Verification.png)
- [Configured NTP peers](../Screenshots/Desktop-Experience/Active-Directory/22-Windows-Time-Synchronization-Verification2.png)
- [DCDIAG Windows Time check passed](../Screenshots/Desktop-Experience/Active-Directory/23-DCDIAG-W32Time-Passed.png)
