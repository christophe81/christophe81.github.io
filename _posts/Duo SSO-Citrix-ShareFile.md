---
layout: post
title: Duo SSO + Citrix ShareFile Configuration Guide
categories: SAML SSO
---

## How to Configure Duo SSO SAML 2.0 for Citrix ShareFile

*All docs on this site are unofficial*

### Prerequisites:
1. You have a [ShareFile](https://sharefile.com) account that supports SAML authentication
1. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source

### Outline
1. What's been tested
1. SSO Configuration Steps

### What's been tested:

I have tested the following and confirmed they work:
1. SP-initiated authentication
1. IdP-initiated authentication


### SSO Configuration Steps

For more information on configuring SAML for Citrix ShareFile, see [How to Configure Single Sign-On (SSO) for ShareFile](https://support.citrix.com/article/CTX208557)

#### Create a Citrix ShareFile application in Duo
1. Login to your Duo Admin Panel
2. Navigate to **Protect an Application**
3. Search for **Generic Service Provider** and click **Protect** Note: be sure to select the one for Single Sign-On (hosted by Duo)

Note: Leave your Duo Admin panel open on this newly created application as you need to copy items from it into your ShareFile account.

#### Configure SSO in Citrix ShareFile
1. Login to your ShareFile account as an administrator with access to modify SAML settings. 
2. Navigate to **Settings - Admin Settings - Security - Login & Security Policy - Single sign-on/SAML 2.0 Configuration** 
3. Under Basic Settings configure the following: 
   1. **Enable SAML** select **Yes**
   2. **ShareFile Issuer / Intity ID:** should be https://[yoursharefilesubdomain].sharefile.com/saml/info If you are using a ShareFile account on .eu, change .com to .eu
   3. **Your IDP Issuer / Entity ID:** is the **Entity ID** of the new Citrix ShareFIle application you created in Duo. Eample https://[guid].sso.duosecurity.com/saml2/sp/[GUID}/metadata
