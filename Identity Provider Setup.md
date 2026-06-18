# PCUK – Toca Identity Provider Setup

## 🔐 1. Entra App Registration Setup

### Prerequisites
- Global Admin / Application Administrator permissions
- Access to existing **Toca app registration**

### Steps

1. Go to Entra Admin Center  
   https://entra.microsoft.com

2. Navigate:
   App registrations → Search **"toca"** → Select **"Toca-pcuk"**

3. From **Overview**
   - Copy:
     - Application (Client) ID
   - Create / confirm:
     - Client Secret *(only visible at creation — store securely)*

4. Go to **Authentication**
   - Ensure **Redirect URIs** match:
     - https://github.com/tocalabs/toca-docs/blob/main/src/admin/identity_providers.md

5. Go to **API Permissions**
   - Confirm required permissions:
     - Microsoft Graph → Mail.Read
   - Click:
     - ✅ Grant admin consent

---

## 👤 2. Enterprise Application Access

1. Navigate:
   Enterprise applications → All applications → **"Toca-pcuk"**

2. Go to:
   Users and groups

3. Add required user/service account:
   - Example:
     - `msgbroker2`

> ⚠️ This controls who the application can sign in as (runtime access)

---

## 🌐 3. Toca Identity Provider Configuration

1. Go to:
   https://prostatecanceruk.toca.cloud/

2. Navigate:
   Admin settings → Identity providers → **TOCA-PCUK**

3. Configure:

- Client ID → From Entra App Overview  
- Client Secret → From Entra  
- Auth Endpoint → From toca-docs  
- Token Endpoint → From toca-docs  

---

## 🔗 4. Datastore Identity Binding

1. Navigate:
   Datastore → Auth

2. Click:
   Add identity

3. Select:
   - Identity Provider → **TOCA-PCUK**

4. Sign in using:
   - Target mailbox account (e.g. `msgbroker2`)

---

## ✅ Key Notes

- App Registration = configuration (IDs, secrets, permissions)
- Enterprise Application = access control (who can use it)
- Identity Provider in Toca = connection setup
- Datastore Identity = actual authenticated session

---

## ⚠️ Gotchas / Common Issues

### Client Secret
- Only visible once on creation  
- If lost → must regenerate and update in Toca

### Identity Not Working
- Ensure identity is added in:
  - Datastore → Auth
- Without this, the provider is not actively usable

---

## 🧠 Summary

This setup connects a Microsoft Entra app registration to Toca using an identity provider, enabling secure authenticated access to a mailbox via a service account.
