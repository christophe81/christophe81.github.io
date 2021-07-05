---
layout: post
title: Duo SSO + Citrix Workspace Configuration Guide
categories: SAML SSO
excerpt_separator: <!--more-->
---

This page is under construction, full details of the configuration _**COMING SOON**_

## How to Configure Duo SSO SAML 2.0 for Citrix Workspace

*All docs on this site are unofficial* 

### Prerequisites:
1. You have a [Citrix Workspace](https://www.citrix.com/products/citrix-workspace/) account
  1. Note, Citrix requires Active Directory attributes to be passed to them during the federated login flow. While it's possible to configure Duo SSO with a SAML IdP, such as Azure AD or Okta, you must make sure to either sync those SAML IdPs with your local AD or confirm that the attributes listed on [Citrix's Documentation page under **Active Directory**](https://docs.citrix.com/en-us/citrix-cloud/citrix-cloud-management/identity-access-management/saml-identity.html) are passed to Duo
3. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source

### Outline
1. What's been tested
2.  SSO Configuration Steps

### What's been tested:

I have tested the following and confirmed they work:
1. SP-initiated authentication
2. [Duo Passwordless](https://duo.com/solutions/passwordless)

### SSO Configuration Steps

For more information on configuring SAML for Citrix Workspace, see [Connect SAML as an identity provider to Citrix Cloud (Technical Preview)](https://docs.citrix.com/en-us/citrix-cloud/citrix-cloud-management/identity-access-management/saml-identity.html)


