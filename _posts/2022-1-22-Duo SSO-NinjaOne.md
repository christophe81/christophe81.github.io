---
layout: post
title: Duo SSO + NinjaOne Configuration Guide
categories: SAML SSO
excerpt_separator: <!--more-->
--- 

## How to Configure Duo SSO SAML 2.0 for NinjaOne

*All docs on this site are unofficial*

### Prerequisites:
1. You have a [NinjaOne](https://ninjaone.com) account
1. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source

### Outline
1. What's been tested
1. SSO Configuration Steps

<!--more-->

### What's been tested:

I have tested the following and confirmed they work:
1. SP-initiated authentication
1. [Duo Passwordless](https://duo.com/solutions/passwordless)

### SSO Configuration Steps

#### Create a NinjaOne application in Duo
1. Login to your Duo Admin Panel
2. Navigate to **Protect an Application**
3. Search for **Generic Service Provider** and click **Protect** Note: be sure to select the one for Single Sign-On (hosted by Duo)
4. Copy the **Metadata URL** OR next to **SAML Metadata** click **Download XML** and save the file to your desktop
5. Do not close out of the NinjaOne admin console as we will be returning to finish the configuration shortly

#### Configure SSO for NinjaOne
1. Login to your NinjaOne account as an administrator
2. Click **Configuration** 
3. Navigate to **Accounts - Single Sign-On (Beta)** then click **Configure**
4. Copy the **SP Identifier (Entity ID)** and **Assertion Reply URL** URLs

#### Configure the NinjaOne application in Duo
1. Navigate back to your new Generic Service Provider application within the Duo Admin pane
2. Next to **Service Provider Name** input **NinjaOne**
3. Next to **Entity ID** input the **SP Identifier (Entity ID)** you just grabbed from your NinjaOne admin console. It shoudl be something similar to: https://app.ninjarmm.com/ (make sure to include the trailing /)
4. Next to **Assertion Consumer Service** input the **Assertion Reply URL** you copied from the NinjaOne admin console. It should be something similar to: https://app.ninjarmm.com/ws/account/saml-login
6. **NameID format** should stay as the default: urn.oasis:names:tc:SAML:1.1:nameid-format:emailAddress
7. Next to **NameID attribute** input the attribute that maps to your email address. If possible, I always recommend choosing Duo's preconfigured attributes, in this case <Email Address> This will allow you to change Duo SSO Authentication Source in the future, if needed. For example, from AD to a SAML IdP. 
8. Next to **Signing options** leave both **Sign response** and **Sign assertion** checked.
9. Scroll down to the **Policy** section and choose the policy you wish to implement for this application. If you created a Passwordless policy, be sure to select that now if you want Passwordless authentication into NinjaOne.
10. Scroll down to the **Settings** section and next to **Name** add NinjaOne. You may also want to configure other options under this section, depending on how you have Duo MFA configured for your users.
11. Scroll to the bottom and click **Save**

Return to the NinjaOne Admin Console

#### Finish the NinjaOne SAML configuration
1. From the Single Sign-One (Beta) configuration page, click **File**, **URL**, or **</>XML**. In my testing I choose **URL**
2. Add the **Metadata URL** or **File** from which you received from the Duo SSO application we just created.
3. Click **Test** and perform an authentication.
4. After a successful authentication click **Save**
5. To enable SAML authentication right away, click **Enable Now**
  
You are now ready to use Duo SSO for SAML authentication into NinjaOne. Note, today SAML authentication is only avaiable for Administrators and Technicians within the NinjaOne Website. It is not available for End Users or the NinjaOne mobile application.
