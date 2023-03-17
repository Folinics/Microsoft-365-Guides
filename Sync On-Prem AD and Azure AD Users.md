# Sync On-Prem AD and Azure AD Users
This usually occurs when you change the user logon name on an on-prem Active Directory, or if you have identical SMTP addresses under the `proxyAddresses` attribute for multiple users. 
What happens is a duplicate Azure AD user ends up getting created and falls out-of-sync with the on-prem user. This guide aims to correct that.










## Credits
The bulk of this guide is from https://aidenwebb.com/posts/how-to-hard-link-azure-ad-connect-on-prem-users-to-azure-ad-office-365-accounts/, but with corrections and improvements.
