# Identity and Account Administration

## Overview

This part of the lab focuses on common Active Directory administration tasks: organizing directory objects, managing user and group membership, provisioning test accounts, and applying password policies.

## Organizational units and administrative account

A dedicated `_ADMINS` Organizational Unit was created to separate administrative accounts from standard user objects. A separate administrative account (`a-jperalta`) was created and added to the `Domain Admins` security group.

The account was used to validate domain sign-in using both the UPN and NetBIOS domain formats. The `Domain Admins` membership is used only for the isolated lab; broad privileged group membership should be minimized in production environments.

Evidence:

- [Administrative OU](../Screenshots/Desktop-Experience/Active-Directory/14-Adding-OUs.png)
- [User created in the administrative OU](../Screenshots/Desktop-Experience/Active-Directory/15-Adding-User-To-OU.png)
- [Account creation options](../Screenshots/Desktop-Experience/Active-Directory/16-New-User-Password-Options.png)
- [Domain Admins membership](../Screenshots/Desktop-Experience/Active-Directory/17-Adding-User-To-Group.png)
- [Domain administrator sign-in](../Screenshots/Desktop-Experience/Active-Directory/18-Domain-Administrator-Sign-In.png)

## User, OU, and group administration

The lab exercises included:

- Creating and organizing Organizational Units.
- Creating and modifying domain user accounts.
- Creating security groups and managing their membership.
- Adding users to and removing users from security groups.
- Practicing account lifecycle tasks, including disabling and deleting test accounts.
- Verifying group membership and account placement in the directory.

## Bulk account provisioning

A PowerShell workflow was used to bulk-create more than 1,000 test user accounts in the lab directory. AI tools assisted with developing and refining the workflow; the generated commands were used in the isolated test environment.

The source archive supplied for this portfolio did not include the standalone account-provisioning `.ps1` file or a run log. The accomplishment is therefore summarized here, but the script itself has not been reconstructed or represented as an original source file. A reviewed, parameterized script and sanitized sample input/output would strengthen this evidence when added.

## Password policy administration

Fine-Grained Password Policies were configured using Password Settings Objects (PSOs). The lab work included creating policy objects, configuring password and account lockout settings, assigning policies to appropriate users or security groups, and checking policy application.

The supplied screenshot set does not contain dedicated PSO configuration captures. The work is included as a completed lab exercise based on the project information provided, but policy-specific screenshots or sanitized PowerShell output should be added as supporting evidence in a later update.

## Administrative command examples

```powershell
# Inspect the groups associated with a user
Get-ADPrincipalGroupMembership -Identity "a-jperalta" |
    Select-Object Name

# List domain organizational units
Get-ADOrganizationalUnit -Filter * |
    Select-Object Name,DistinguishedName

# List enabled and disabled user accounts
Get-ADUser -Filter * -Properties Enabled |
    Select-Object Name,SamAccountName,Enabled
```

These are reference commands for inspecting directory objects. Review identity scope and permissions before running administrative commands in any environment.

## Skills demonstrated

- Active Directory object organization and administration.
- User and security-group lifecycle management.
- Administrative account separation and privileged group awareness.
- PowerShell-assisted bulk provisioning.
- Fine-Grained Password Policy administration.
- Basic validation of directory objects and group membership.
