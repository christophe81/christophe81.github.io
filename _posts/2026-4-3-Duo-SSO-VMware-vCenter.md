---
layout: post
title: Duo SSO + VMware vCenter Server Configuration Guide
categories: OIDC SSO
excerpt_separator: <!--more-->
---

## How to Configure Duo SSO (OAuth 2.1 / OIDC) for VMware vCenter Server Identity Federation

*All docs on this site are unofficial*

### Prerequisites:
1. You have a [VMware vCenter Server](https://www.vmware.com/products/vcenter.html) version 7.0 or later
1. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source
1. vCenter Server can reach `sso-*.sso.duosecurity.com` on port 443
1. Consistent DNS/FQDN resolution between vCenter and Duo SSO endpoints

### Outline
1. What's been tested
1. Authentication flow overview
1. SSO Configuration Steps
1. User provisioning
1. Assigning vSphere permissions
1. Troubleshooting

<!--more-->

### What's been tested:

This guide covers the OIDC-based Identity Provider Federation flow. The configuration approach below is derived from community-tested guides for non-listed OIDC providers (Authentik, Keycloak, Kanidm, Zitadel) that successfully integrate with vCenter using this same method.

Sources referenced:
- [Duo SSO OAuth 2.1 / OIDC documentation](https://duo.com/docs/sso-oauth-server)
- [vCenter Identity Federation with Authentik](https://williamlam.com/2025/01/vcenter-server-identity-federation-with-authentik-identity-provider.html) — William Lam, January 2025
- [vCenter Identity Federation with Keycloak (without SCIM)](https://williamlam.com/2025/01/vcenter-server-identity-federation-with-keycloak-identity-provider-without-scim.html) — William Lam, January 2025
- [vCenter Identity Federation with Kanidm](https://williamlam.com/2025/04/vcenter-server-identity-federation-with-kanidm.html) — William Lam, April 2025
- [vCenter Identity Federation with Zitadel](https://williamlam.com/2025/04/vcenter-server-identity-federation-with-zitadel.html) — William Lam, April 2025

### How the Authentication Flow Works

1. A user navigates to the vSphere Client login page and selects the federated identity login option
2. vCenter redirects to Duo SSO's authorization endpoint
3. The user authenticates via Duo (primary credentials + MFA via Universal Prompt)
4. Duo redirects back to vCenter with an authorization code
5. vCenter exchanges the code for tokens and establishes a session

> **Note:** Authentication (AuthN) is handled by Duo SSO, but authorization (AuthZ) is still managed by vCenter Server. After configuring federation and provisioning users, you must assign vSphere permissions (roles) to those users or groups within vCenter.

### SSO Configuration Steps

#### Create a vCenter Application in Duo

1. Login to your [Duo Admin Panel](https://admin.duosecurity.com)
2. Navigate to **Applications → Application Catalog**
3. Search for **OAuth 2.1 / OIDC - Single Sign-On** (labeled "SSO") and click **+ Add**
4. Give it a descriptive name (e.g., "VMware vCenter Server")

#### Create a Client

1. Under the new application, click **Add Client**
2. Enter a client name (e.g., "vCenter - prod")
3. Duo will generate a **Client ID** and **Client Secret** — save both securely. The Client Secret is only shown once (it can be reset later via **Reset Client Secret**)

#### Configure the Client

**Grant Types:**

Enable **Authorization Code** — this is the OAuth flow that vCenter uses for identity federation. Leave PKCE optional unless you have confirmed your vCenter version supports it.

**Redirect URI:**

Enter the vCenter redirect URI in this format:

```
https://<VCENTER_FQDN>/federation/t/CUSTOMER/auth/response/oauth2
```

Replace `<VCENTER_FQDN>` with your vCenter Server's fully qualified domain name. You will confirm the exact redirect URI from vCenter's SSO configuration screen in a later step.

**Scopes:**

Ensure the following scopes are available to the client:

| Scope | Required | Purpose |
|-------|----------|---------|
| `openid` | Yes | Core OIDC — required for ID token issuance |
| `profile` | Yes | Provides `name`, `given_name`, `family_name` claims |
| `email` | Yes | Provides `email` claim |

**Token Lifetime:**

The default access token lifetime of 60 minutes works well for vCenter sessions. Refresh tokens are optional and not required for the vCenter federation flow.

**User Access:**

Under the application's **User Access** settings, choose which Duo groups can access this application, or allow all users. Ensure the users who need vCenter access are included.

#### Record the OIDC Metadata

From the application's **Metadata** section, note the following values. You'll enter these into vCenter in the next section.

| Value | Where to Find It | Example |
|-------|-------------------|---------|
| **Client ID** | Metadata section | `DIABC123678901234567` |
| **Client Secret** | Shown at client creation | *(save securely)* |
| **Issuer URL** | Metadata section | `https://sso-abc1def2.sso.duosecurity.com/oidc/DIABC123678901234567` |
| **OpenID Configuration URL** | Issuer URL + path | `https://sso-abc1def2.sso.duosecurity.com/oidc/DIABC123678901234567/.well-known/openid-configuration` |

The OpenID Configuration URL is the discovery endpoint that vCenter uses to auto-discover all other endpoints (authorize, token, JWKS, userinfo).

#### Configure vCenter Server — Open Identity Provider Configuration

1. Log in to the vSphere Client as `administrator@vsphere.local`
2. Navigate to **Administration → Single Sign-On → Configuration**
3. Under **Identity Provider**, click **Change Provider**

#### Select the Provider Type

vCenter's identity federation wizard lists several named IdP options. Since Duo is not listed by name, select one of the generic-compatible options:

- **PingFederate**
- **Okta**

> **Note:** These dropdown labels are cosmetic — the underlying federation mechanism is the same standard OIDC flow regardless of which named option you select. William Lam's community guides for Authentik, Keycloak, Kanidm, and Zitadel all use the PingFederate or Okta option to connect non-listed OIDC providers to vCenter.

#### Enter Directory Information

| Field | Value |
|-------|-------|
| **Directory Name** | A friendly name (e.g., "Duo SSO") |
| **Domain Name(s)** | The email domain of your users (e.g., `company.com`) |

> **Important:** Do not use the same domain name as vCenter's own SSO domain (`vsphere.local`). The domain(s) you enter here must match the email domain of the users being federated. Using a conflicting domain will cause user sync errors.

#### Enter OIDC Configuration

| vCenter Field | Duo Value |
|---------------|-----------|
| **Identity Provider Name** | A friendly name (e.g., "Duo SSO") |
| **Client Identifier** | The **Client ID** from Duo |
| **Shared Secret** | The **Client Secret** from Duo |
| **OpenID Address** | The **OpenID Configuration URL** from Duo |

#### Confirm the Redirect URI

After entering the OIDC details, vCenter displays a **Redirect URI**. Copy this value and verify it matches what you entered in the Duo application. If it differs, update the redirect URI in your Duo application to match exactly.

The expected format is:
```
https://<VCENTER_FQDN>/federation/t/CUSTOMER/auth/response/oauth2
```

#### TLS Certificate Chain

If vCenter prompts for a certificate chain, provide the full TLS certificate chain for Duo's SSO endpoints. You can retrieve this using `openssl`:

```
openssl s_client -connect sso-abc1def2.sso.duosecurity.com:443 \
  -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM
```

Replace the hostname with your Duo SSO endpoint from the Issuer URL.

### User Provisioning

After the OIDC federation is configured, vCenter displays a **SCIM tenant URL** and **secret token** for user and group provisioning.

> **Important:** The Duo SSO OAuth 2.1 / OIDC application does not support automated SCIM 2.0 outbound provisioning (per the [Duo SSO OAuth 2.1/OIDC documentation](https://duo.com/docs/sso-oauth-server)). You cannot automatically sync users and groups from Duo into vCenter. Use one of the manual provisioning approaches below.

#### Option A: Manual User Provisioning via vIDB Scripts (Recommended)

vCenter's Identity Broker (vIDB) accepts user records via shell scripts. This is the same approach used to provision users from IdPs without SCIM support such as Keycloak, Kanidm, and Zitadel (per [William Lam's federation guides](https://williamlam.com/2025/01/vcenter-server-identity-federation-with-keycloak-identity-provider-without-scim.html)).

1. **Prepare a CSV file** with the following format:

```
# Username, First Name, Last Name, Email, External Id
jsmith, John, Smith, jsmith@company.com, <DUO_USER_ID>
mjones, Mary, Jones, mjones@company.com, <DUO_USER_ID>
```

The **External ID** is the user's unique identifier in Duo. Find it in the Duo Admin Panel under **Users → [User] → User ID**.

2. **Download the provisioning scripts** from William Lam's GitHub (referenced in his federation blog posts):
  - `manual-scim-sync-users.sh` — publishes users to vIDB
  - `manual-scim-remove-users.sh` — removes users from vIDB

3. **Run the sync script** on the vCenter Server Appliance (VCSA):

```
./manual-scim-sync-users.sh 'administrator@vsphere.local' 'YourPassword' users.csv
```

4. **Repeat** whenever you add or remove users who need vCenter access.

#### Option B: Intermediate SCIM Bridge

If you have a SCIM-capable directory (e.g., Entra ID, Okta) that already syncs with Duo, you can configure that directory's SCIM provisioning to push users and groups to vCenter's SCIM endpoint directly. This is more complex but enables automated provisioning.

### Assigning vSphere Permissions

Federation handles authentication only. You must grant vSphere roles to federated users before they can access resources:

1. In the vSphere Client, navigate to the inventory object where you want to assign permissions (e.g., a datacenter, cluster, or the root vCenter)
2. Go to **Permissions** and click **Add**
3. Change the domain dropdown to the directory name you configured (e.g., "Duo SSO")
4. Search for the provisioned user or group
5. Assign the appropriate vSphere role (e.g., Administrator, Read-Only, etc.)

### Verify SSO

1. Open a new browser window (or incognito session) and navigate to the vSphere Client at `https://<VCENTER_FQDN>/ui`
2. You should see a login option for the federated identity provider alongside the traditional `vsphere.local` login
3. Click the federated login option — you should be redirected to Duo SSO
4. Authenticate with your primary credentials and complete MFA via the Duo Universal Prompt
5. After successful authentication, you should be redirected back to the vSphere Client with an active session

> **Tip:** The `administrator@vsphere.local` account always remains available via the local login option, ensuring you can recover access if federation is misconfigured.

### Troubleshooting

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Redirect fails after Duo login | Redirect URI mismatch | Compare the redirect URI in Duo's client config with what vCenter displays. They must match exactly. |
| "User not found" after authentication | User not provisioned in vIDB | Run the manual SCIM sync script to publish the user to vCenter |
| Certificate error during setup | vCenter can't verify Duo's TLS chain | Import Duo's full certificate chain into vCenter's trust store |
| User authenticates but has no permissions | Authorization not configured | Assign vSphere roles to the federated user or group |
| Login page doesn't show federated option | Federation not fully configured | Verify all OIDC settings are saved and the identity provider is active in the SSO configuration |
| Domain name conflict | Email domain matches `vsphere.local` | Use a distinct domain name that matches your users' actual email domain |

### Reference: Duo SSO OIDC Endpoints

All endpoints are relative to the Issuer URL (`https://sso-{id}.sso.duosecurity.com/oidc/{APP_ID}`):

| Endpoint | Path |
|----------|------|
| Discovery | `/.well-known/openid-configuration` |
| Authorization | `/authorize` |
| Token | `/token` |
| JWKS | `/jwks` |
| UserInfo | `/userinfo` |
| Token Introspection | `/token_introspection` |
