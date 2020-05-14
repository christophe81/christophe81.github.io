---
layout: post
title: Duo SSO + Box Configuration Guide
categories: SAML, SSO
---

## How to Configure Duo SSO SAML 2.0 for Box

### Prerequisites:
1. You have a [Box](https://box.com) Enterprise account
1. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source

### Outline
1. What's been tested
1. SSO Configuration Steps
1. Implementing SAML JIT Provisioning and Group Push

### What's been tested:

I have tested the following and confirmed they work:
1. SP-initiated authentication
1. IdP-initiated authentication
1. Just In Time (JIT) Provisioning
1. SAML Group Push

### SSO Configuration Steps

For more information on configuring SAML for Box, see [Box's Setting up Single Sign On (SS) for your Enterprise](https://support.box.com/hc/en-us/articles/360043696514-Setting-Up-Single-Sign-On-SSO-for-your-Enterprise)

### Create a Box application in Duo
1. Login to your Duo Admin Panel
2. Navigate to **Protect an Application**
3. Search for **Generic Service Provider** and click **Protect** Note: be sure to select the one for Single Sign-On (hosted by Duo)
4. Next to **SAML Metadata** click **Download XML** and see the file to your desktop

#### Configure SSO for Box
1. Login to your Box account as a primary administrator
2. Click **Admin Console** 
3. Navigate to **Enterprise Settings - User Settings - Configure Single Sign On (SSO) for All Users**, then click **Configure**
4. Since Duo is not part of the **Identity Providers** dropdown, I selected ADFS instead. Note, you can choose to follow the **I don't see my provider...** but that does mean some configuration options below will be different, specifically for JIT Provisioning and SAML Group Push.
5. Click **Choose File** and select your Duo Generic Service Provider XML file and click **Submit**
6. Box will process your metadata file which can take up to 24 hours. Luckily they will email you once everything is ready so you don't have to continue checking in.
7. Once you have been notified by Box that everything is ready, you will have the option to enable SSO. I recommend starting with enabling **SSO Test Mode**. This will allow you to test your configuration without requiring SSO authentication. 

### Configure the Box application in Duo
1. Navigate back to your new Generic Service Provider application within the Duo Admin panel.
2. Next to **Service Provider Name** input **Box**
3. Next to **Entity ID** input box.net
4. Next to **Assertion Consumer Service** input **https://sso.services.box.net/sp/ACS.saml2**
5. Scroll down to the  **SAML Response** section
6. Next to **NameID attribute** input the attribute that maps to your email address. If possible, I always recommend choosing Duo's preconfigured attributes, in this case <Email Address> This will allow you to change Duo SSO Authentication Source in the future, if needed. For example, from AD to a SAML IdP. 
7. Next to **Signing options** leave both **Sign response** and **Sign assertion** checked.
8. Scroll down to the **Policy** section and choose the policy you wish to implement for this application.
9. Scroll down to the **Settings** section and next to **Name** add Box. You may also want to configure other options under this section, depending on how you have Duo MFA configured for your users.
10. Scroll to the bottom and click **Save**

You are now ready to test SSO authentication into Box!

#### Configure Just In Time (JIT) Provisioning
1. Navigate to your Box application within the Duo Admin Panel
2. Scroll down to the **SAML Response** section
3. Next to **Map Attributes** add the following. Note: again I recommend using Duo's preconfigured attributes, if possible:
   1. IdP Attribute: First Name & SAML Response Attribute: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
   2. IdP Attribute: Last Name &  SAML Response Attribute: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
   3. IdP Attribute: Email Address &  SAML Response Attribute:http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
4. Contact your Box representative and let them know you would like for them to enable SSO Auto Provisioning. In your correspondence you will need to let them know the attributes created above.

#### Configure SAML Group Push
1. Contact your Box representative and let them know you would like for them to enable SAML Group Push. Once enabled login into your Box account and navigate to **Enterprise Settings - User Settings - User Groups Settings**. 
2. Check the box next to the options you wish to enable. Note, I enabled all three options: **Add new groups upon SSO user login**, **Add users to groups upon SSO user login** and **Remove user from groups upon SSO user login**.
3. Navigate to your Box application within the Duo Admin Panel
4. Scroll down to the **SAML Response** section
5. Next to **Role Attributes** add the following:
   1. Attribute Name: http://schemas.xmlsoap.org/claims/Group
   2. Service Provider's Role: (the name of the group you want created in Box)
   3. Duo Groups: (the Duo group you want to have populated in the Box group)
6. Contact your Box representative and let them know you have configured SAML Group Push as they need to update the Groups SAML attribute on their end.
7. Test away!

Note: When mapping Box Groups to Duo Groups, they must be done in a 1:1 relationship. You may create more than one row within the **Role Attributes** section BUT the groups must remain 1:1


