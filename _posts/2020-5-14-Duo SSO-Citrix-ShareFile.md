---
layout: post
title: Duo SSO + Citrix ShareFile Configuration Guide
categories: SAML SSO
excerpt_separator: <!--more-->
---

Since this creation of this post, Duo has added ShareFile to their Application Catalog and provided detailed [setup directions](https://duo.com/docs/sso-sharefile).

## How to Configure Duo SSO SAML 2.0 for Citrix ShareFile

*All docs on this site are unofficial* 

### Prerequisites:
1. You have a [ShareFile](https://sharefile.com) account that supports SAML authentication
1. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source

### Outline
1. What's been tested
1. SSO Configuration Steps

<!--more-->

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
4. Make a point of where the **Entity ID**, **Single Sign-On URL**, **Signle Sign-On Logout URL** are located as you will need to copy them into ShareFile in the next section.
5. Download the **Certificate** and save it to your desktop.

Note: Leave your Duo Admin panel open on this newly created application as you need to copy items from it into your ShareFile account.

#### Configure SSO in Citrix ShareFile
1. Login to your ShareFile account as an administrator with access to modify SAML settings. 
2. Navigate to **Settings - Admin Settings - Security - Login & Security Policy - Single sign-on/SAML 2.0 Configuration** 
3. Under Basic Settings configure the following: 
   1. **Enable SAML** select **Yes**
   2. **ShareFile Issuer / Entity ID:** should be **Entity ID** found in Duo. Example: [https://[yoursharefilesubdomain].sharefile.com/saml/info] If you are using a ShareFile account on .eu, change .com to .eu
   3. **Your IDP Issuer / Entity ID:** is the **Entity ID** of the new Citrix ShareFile application you created in Duo. Example [https://[guid].sso.duosecurity.com/saml2/sp/[GUID]/metadata]
   4. **X.509 Certificate** upload the certificate that you downloaded from Duo in the section above
   5. **Login URL** should be the **Single Sign-On URL** found in Duo. Example: [https://GUID.sso.duosecurity.com/saml2/sp/GUID/sso]
   6. **Logout URL** should be the **Signle Sign-On Logout URL** found in Duo. Example: [https://GUID.sso.duosecurity.com/saml2/sp/GUID/slo]
4. I left all **Optional Settings** as the defaults
5. Click **Save**

#### Configure the ShareFile application in Duo
1. Navigate back to the application we created above in your Duo Admin panel. 
2. Next to **Service Provider name** enter anything. I choose **ShareFile**
3. Next to **Entity ID** enter the **Entity ID** found in ShareFile. Example: [https://chris.sharefile.com/saml/info]
4. Next to **Assertion Consumer Service** enter the following. In an attempt to explain this URL construction, it is your ShareFile URL/saml/acs?idpentityid= the Duo Entity ID but without the /metadata. Example:  [https://[yoursharefilesubdomain].sharefile.com/saml/acs?idpentityid=https://[guid].sso.duosecurity.com/saml2/sp/[GUID]]
5. **NameID format** should stay as the default: urn.oasis:names:tc:SAML:1.1:nameid-format:emailAddress
6. Next to **NameID attribute** input the attribute that maps to your email address. If possible, I always recommend choosing Duo's preconfigured attributes, in this case This will allow you to change Duo SSO Authentication Source in the future, if needed. For example, from AD to a SAML IdP.
7. Next to **Signing options** leave both **Sign response** and **Sign assertion** checked.
8. Scroll down to the **Policy** section and choose the policy you wish to implement for this application.
9. Scroll down to the **Settings** section and next to **Name** add ShareFile. You may also want to configure other options under this section, depending on how you have Duo MFA configured for your users.
10. Scroll to the bottom and click **Save**

You are now ready to test SSO authentication into ShareFile! To test IdP initiated authentication, copy the **Single Sign-On URL** from the ShareFile application in your Duo Admin panel. If you want to test SP initiated authentication, use the following: [https://[yoursharefilesubdomain].sharefile.com/saml/login]
