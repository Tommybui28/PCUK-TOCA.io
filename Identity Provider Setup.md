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
   - Target mailbox account (e.g. `msgbroker2`), **or**
   - An admin account that has **Full Access** delegation on the target mailbox (see Section 5)

---

## 📬 5. Mailbox Delegation (Admin Account Sign-In)

If you sign in to the Toca identity using an **admin account** (e.g. `tb_sp_admin`) instead of the mailbox account itself, the admin must be granted **Full Access** delegation on each target mailbox.

> ⚠️ **Global Admin role does NOT grant mailbox access.** Entra/Graph permissions and Exchange mailbox permissions are separate. Without delegation, Graph will return:  
> `404 ErrorItemNotFound — Default folder AllItems not found`

### Option A — Exchange Admin Center
1. Go to https://admin.exchange.microsoft.com
2. **Recipients → Mailboxes → [target mailbox, e.g. msgbroker3]**
3. **Delegation → Read and manage (Full Access) → Edit**
4. **+ Add members** → add the admin account → **Save**

### Option B — PowerShell
```powershell
Connect-ExchangeOnline -UserPrincipalName <admin-upn>

Add-MailboxPermission -Identity msgbroker3@prostatecanceruk.org `
    -User tb_sp_admin@prostatecanceruk.org `
    -AccessRights FullAccess `
    -InheritanceType All `
    -AutoMapping $false
```

> 💡 `-AutoMapping $false` prevents the mailbox auto-appearing in the admin's Outlook profile.

Propagation typically takes **15–60 minutes**. The cached token in Toca may also need to refresh (disconnect/reconnect the identity to force this).

---

## 🌍 6. API Caller URL Targeting

API Callers **must target the specific mailbox by UPN**, not the `/me/` endpoint.

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `https://graph.microsoft.com/v1.0/me/messages` | `https://graph.microsoft.com/v1.0/users/msgbroker3@prostatecanceruk.org/messages?$top=20` |

### Why
- `/me/messages` resolves to the **signed-in identity's own mailbox** — if the identity was authenticated as an admin, it pulls the admin's inbox, not the target mailbox.
- `/users/{upn}/messages` explicitly targets the intended mailbox, and works correctly provided the signed-in identity has Full Access delegation (per Section 5).

---

## ✅ Key Notes

- App Registration = configuration (IDs, secrets, permissions)
- Enterprise Application = access control (who can use it)
- Identity Provider in Toca = connection setup
- Datastore Identity = actual authenticated session
- Mailbox Delegation = Exchange-level permission required when signing in as a non-owner
- API Caller URL = must target the mailbox UPN explicitly

---

## ⚠️ Gotchas / Common Issues

### Client Secret
- Only visible once on creation  
- If lost → must regenerate and update in Toca

### Identity Not Working
- Ensure identity is added in:
  - Datastore → Auth
- Without this, the provider is not actively usable

### 404 `ErrorItemNotFound` from Graph
- Cause: signed-in identity has no mailbox access on the target mailbox
- Fix: grant **Full Access** delegation (see Section 5), wait for propagation, refresh the Toca identity token

### API Caller returns the wrong inbox
- Cause: URL is using `/me/messages` instead of `/users/{upn}/messages`
- Fix: target the mailbox UPN directly (see Section 6)

### Admin role ≠ Mailbox access
- Being Global Admin in Entra does **not** grant Exchange mailbox rights
- Mailbox access must be granted explicitly via Exchange delegation

---

## 🧠 Summary

This setup connects a Microsoft Entra app registration to Toca using an identity provider, enabling secure authenticated access to a mailbox via a service account.

When using an admin account (rather than the mailbox account itself) to authenticate the Toca identity, the admin must also be granted **Full Access mailbox delegation** in Exchange, and API Callers must target the specific mailbox UPN — not `/me/`.
