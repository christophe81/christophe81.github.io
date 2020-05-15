---
layout: post
title: Duo SSO + GitHub Enterprise Configuration Guide
categories: SAML SSO
excerpt_separator: <!--more-->
---

## How to Configure Duo SSO SAML 2.0 for GitHub Enterprise

*All docs on this site are unofficial* 

### Prerequisites:
1. You have a [GitHub Enterprise](https://github.com/enterprise) account
1. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source

### Outline
1. What's been tested
1. SSO Configuration Steps

<!--more-->

### What's been tested:

I have tested the following and confirmed they work:
1. SP-initiated authentication
1. IdP-initiated authentication
1. Just In Time (JIT) provisioning

### SSO Configuration Steps

For more information on configuring SAML for GitHub Enterprise, see [Enabling and testing SAML single sign-on for your organization](https://help.github.com/en/github/setting-up-and-managing-organizations-and-teams/enabling-and-testing-saml-single-sign-on-for-your-organization)

#### Create a GitHub Enterprise application in Duo
1. Login to your Duo Admin Panel
2. Navigate to **Protect an Application**
3. Search for **Generic Service Provider** and click **Protect** Note: be sure to select the one for Single Sign-On (hosted by Duo)
4. Make a point of where the **Entity ID** and **Single Sign-On URL** are located as you will need to copy them into GitHub in the next section.
5. Download the **Certificate** and save it to your desktop.

Note: Leave your Duo Admin panel open on this newly created application as you need to copy items from it into your GitHub account.

#### Configure SSO in GitHub Enterprise
1. Login to your GitHub account with someone who has access to modify SAML settings. 
2. Navigate to **Settings - Security - SAML single sign-on*
3. Check the box next to **Enable SAML Authentication**
4. **Sign on URL** is the **Single Sign-On URL** of the newly created GitHub application in Duo:: Example [https://GUID.sso.duosecurity.com/saml2/sp/GUID/sso]
5. **Issuer** is the **Entity ID** of the newly created GitHub application in Duo:: Example [https://GUID.sso.duosecurity.com/saml2/sp/GUID/metadata]
4. **Public certificate** add the certificate that you downloaded from Duo in the section above. Note: you will need to open the certificate so you can copy and paste it in GitHub. Be sure to include -----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----

Do not click **Test SAML configuration** yet as you need to finish creating the GitHub application in your Duo admin panel. Keep this page open though as you will need to come back once you finish configuring the GitHub application in Duo.

#### Configure the GitHub application in Duo
1. Navigate back to the application we created above in your Duo Admin panel. 
2. Next to **Service Provider name** enter anything. I choose **GitHub**
3. Next to **Entity ID** enter [https://github.com/orgs/[YourGitHubOrgName]]
4. Next to **Assertion Consumer Service** enter [https://github.com/orgs/[YourGitHubOrgName]/saml/consume]
5. **NameID format** should stay as the default: urn.oasis:names:tc:SAML:1.1:nameid-format:emailAddress
6. Next to **NameID attribute** input the attribute that maps to your email address. If possible, I always recommend choosing Duo's preconfigured attributes, in this case This will allow you to change Duo SSO Authentication Source in the future, if needed. For example, from AD to a SAML IdP.
7. Next to **Signing options** leave both **Sign response** and **Sign assertion** checked.
8. Scroll down to the **Policy** section and choose the policy you wish to implement for this application.
9. Scroll down to the **Settings** section and next to **Name** add GitHub Enterprise. You may also want to configure other options under this section, depending on how you have Duo MFA configured for your users.
10. Scroll to the bottom and click **Save**

#### Finish the GitHub configuration
1. Click **Test SAML Configuration** This will take you through the full SAML SSO flow. 
2. One your test is successfully completed, click **Save**.

Optionally you can click **Require SAML SSO authentication for all members of the *YourGitHubOrgName* organization.

You are now ready to use SAML SSO authentication into GitHub Enterprise! To test IdP initiated authentication, copy the **Single Sign-On URL** from the GitHub Enterprise application in your Duo Admin panel. If you want to test SP initiated authentication, use the following: [https://github.com/orgs/[YourGitHubOrgName]/sso]
