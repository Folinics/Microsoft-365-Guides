# Fix Desynced On-Prem AD and Azure AD Users
This usually occurs when you change the user logon name on an on-prem Active Directory, or if you have identical SMTP addresses under the `proxyAddresses` attribute for multiple users.
What happens is a duplicate Azure AD user ends up getting created and falls out-of-sync with the on-prem user. This guide details how to correct the desynchronization.

#### Pre-requisites
**(If applicable)** *On your on-prem AD server, correct any duplicate primary emails, or alternately, SMTPs under the `proxyAddresses` attribute of the affected users. Wait for the next Azure AD connect sync, or force using the `Start-ADSyncSyncCycle -PolicyType Delta` PS command on the host with the Azure AD connect sync service.*


**You will need to be on a host with ADUC. For me, the server that hosts Azure AD connect is also a domain controller, so I performed these steps there.**  

For this tutorial, JohnDoe@YourDomain.com will be the `UserPrincipalName` that is the original user. A duplicate of this user was created in Azure AD after the user logon was changed to John@YourDomain.com in on-prem AD.  

1. Install required PowerShell modules.
```PowerShell
Install-Module -Name MSOnline
# Recommended Extras:
Install-Module -Name AzureAD
Install-Module -Name AZ
```

#### Resolution Steps
1. Connect to Azure Online. This will open a new window, prompting for credentials. This user will need to be an Azure AD / Exchange admin.
```PowerShell
Connect-MsolService
```
2. Get the GUID of your on-prem user. *__Note: John is the user logon__*
```PowerShell
$guid = (Get-ADUser -Identity "John").ObjectGUID
```
3. Convert the GUID to the Immutable ID used in Azure AD
```
$immutableID=[System.Convert]::ToBase64String($guid.tobytearray())
```
4. Using the immutable ID, check to see if there is a user with a matching ID in Azure AD. This should be the duplicate user. *__Note: this command should output `UserPrincipalName`, `DisplayName`, and the `isLicensed` of the matching user__*
```PowerShell
Get-MsolUser | Where-Object {$_.immutableid -eq $immutableid}
```
For our example, let's say it outputs the following:  
`UserPrincipalName:` John@YourDomain.com  
`DisplayName:` John Doe  
`isLicensed:` False
#### ⚠️ Warning ⚠️
##### The following commands are dangerous. Exercise caution.

There are 2 different scenarios.
5. 1, if the output of the previous command shows an Azure AD user that is not in use (in other words, is the duplicate user), and, you have no need for it, delete it with the following commands. Note you will be prompted to confirm the changes.
```PowerShell
# 1. Delete user
Get-MsolUser | Where-Object {$_.immutableid -eq $immutableid} | Remove-MsolUser
# 2. Delete from Azure AD recycle bin
Remove-MsolUser -UserPrincipalName John@YourDomain.com -RemoveFromRecycleBin
```
6. Or 2, if the Azure AD user using the Immutable ID is an account that you use, and you don’t want to delete it, instead, set its ImmutableID to `Null`  
```PowerShell
Get-MsolUser | Where-Object {$_.immutableid -eq $immutableid} | Set-MsolUser -ImmutableId $null
```
7. Find the `UserPrincipalName` of the user you want to correct the link for.

8. Set the ImmutableID for the correct AD user.
```PowerShell
Set-MsolUser -UserPrincipalName JohnDoe@YourDomain.com -ImmutableID $immutableid
```
This will link the existing JohnDoe@YourDomain.com user in Azure AD to John@YourDomain.com, changing the logon username.






## Credits
The bulk of this guide is from https://aidenwebb.com/posts/how-to-hard-link-azure-ad-connect-on-prem-users-to-azure-ad-office-365-accounts/, but with corrections and improvements.  

#### Other Resources  
http://terenceluk.blogspot.com/2020/10/attempting-to-set-immutableid-for-user.html  
https://learn.microsoft.com/en-us/powershell/module/msonline/connect-msolservice?view=azureadps-1.0  
https://learn.microsoft.com/en-us/powershell/module/msonline/?view=azureadps-1.0
