# PCUK - Indentity Provider Setup

## Toca-docs URL

https://github.com/tocalabs/toca-docs/blob/main/src/admin/identity_providers.md

## Entra Admin Center Existing Setup

Required: Relevant Admin priveleges

Navigate to entra.microsoft.com

-> App registrations
-> Search 'toca'
-> Toca-pcuk click
  - Overview:
  - Client credentials which contain secret (which is only available to be seen on creation of new App registration
  - Redirect URls matched with requirements located in toca-docs link

-> API Permissions
  - Relevant permissions needed
  - In this case: Microsoft Graph (API) Mail.Read etc.

-> Enterprise apps
-> All applictations
-> Search 'toca'
-> 'Toca-pcuk' Overview 
-> Users and groups
-> Add relevant user for inbox access, 
  - msgbroker2 in this case

Toca.io - https://prostatecanceruk.toca.cloud/

In Admin settings
-> Identity providers
-> TOCA-PCUK
  - Client ID from entra overview
  - Client secret from Entra
  - Token and Auth endpoint from toca-docs
Then in the Datastore
-> Auth
-> Add identity
-> Select identity provider and log in to relevant user/inbox needed.
