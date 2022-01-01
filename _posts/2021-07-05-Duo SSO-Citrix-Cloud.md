---
layout: post
title: Duo SSO + Citrix Cloud Configuration Guide
categories: SAML SSO
excerpt_separator: <!--more-->
---

## How to Configure Duo SSO SAML 2.0 for Citrix Cloud

*All docs on this site are unofficial* 

### Prerequisites:
1. You have a [Citrix Cloud](https://www.citrix.com/products/citrix-cloud/) account
   1. Note, Citrix requires Active Directory attributes to be passed to them during the federated login flow. While it's possible to configure Duo SSO with a SAML IdP, such as Azure AD or Okta, you must make sure to either sync those SAML IdPs with your local AD or confirm that the attributes listed on [Citrix's Documentation page under **Active Directory**](https://docs.citrix.com/en-us/citrix-cloud/citrix-cloud-management/identity-access-management/saml-identity.html) are passed to Duo
2. [Duo SSO](https://duo.com/docs/sso) is already configured with an authentication source
3. This setup will still require the use of [Citrix Federated Authentication Service](https://docs.citrix.com/en-us/xenapp-and-xendesktop/7-15-ltsr/secure/federated-authentication-service.html) for SSO into Citrix Virtual Apps and Desktops

### Outline
1. What's been tested
2. SSO Configuration Steps

<!--more-->

### What's been tested:

I have tested the following and confirmed they work:
1. SP-initiated authentication
2. [Duo Passwordless](https://duo.com/solutions/passwordless)
3. Unfortunately Citrix Cloud does not support IdP-initiated authentication so I added a Bookmark URL to my Duo Central using the SP-initiated authentication URL.

### SSO Configuration Steps

For more information on configuring SAML for Citrix Cloud, see [Connect SAML as an identity provider to Citrix Cloud (Technical Preview)](https://docs.citrix.com/en-us/citrix-cloud/citrix-cloud-management/identity-access-management/saml-identity.html)

#### Create a Citrix Cloud application in Duo
1. Login to your Duo Admin Panel
2. Navigate to **Protect an Application**
3. Search for **Generic Service Provider** and click **Protect** Note: be sure to select the one for Single Sign-On (hosted by Duo)
4. Make a point of where the **Entity ID**, **Single Sign-On URL**, **Signle Sign-On Logout URL** are located as you will need to copy them into ShareFile in the next section.
5. Download the **Certificate** and save it to your desktop.

Note: Leave your Duo Admin panel open on this newly created application as you need to copy items from it into your Citrix Cloud account.

#### Configure SSO in Citrix Cloud
1. Log on to Citrix Cloud as an administrator
2. Navigate to **Identity and Access Management** - **SAML 2.0**.
3. Click the … on the right of SAML 2.0 and click **Connect** to configure SAML authentication.
4. On the Configure SAML page in Citrix Cloud, paste the **Entity ID into the Entity ID field**.
5. Copy the **Single Sign-on URL** from step 4 above and paste the URL into the **SSO Service URL field** in Citrix Cloud.
6. Copy the **Single Log-out URL** from step 4 above and paste the URL into the **Log URL field** in Citrix Cloud.
7. **Click** the **SAML Metadata** Download and open the XML file.
8. From the XML, make note of the **Entity ID** and **Assertion Consumer Service URL** as we will use this when configuring the application in the Duo Admin Panel. Opentionally you can also make note of the **Single Logout Service Http-redirect** URL.
9. Back in the Citrix Cloud admin panel, set the **Binding Mechanism to Http Redirect**
10. Set the **SAML Response to Sign Either Response or Assertion**.
11. Next to **X.509 Certificate**, click the **Upload File link**, browse to the certificate downloaded in the steps above and upload the certificate.
12. Set the **Authentication Context field to Unspecified/Exact**
13. Set the following attribute values in Citrix Cloud: 
      1. Attribute name for User Display Name = displayName
      2. Attribute name for User Given Name = givenName
      3. Attribute name for User Family Name: familyName
      4. Attribute name forSecure identifier (SID) = cip_sid
      5. Attribute name for User Principal Name (UPN) = cip_upn
      6. Attribute name for Email = cip_email
      7. Attribute name for AD Object Identifier (OID) = cip_oid
 14. We'll be back in the Citrix Cloud admin UI in just a few minutes

#### Configure the Citrix Cloud application in Duo
1. Navigate back to the application we created above in your Duo Admin panel. 
2. Next to **Service Provider name** enter anything. I choose **Citrix Workspace**
3. Next to **Entity ID** enter the **Entity ID** found in Citrix Cloud XML file (step 8 above). Example: [https://saml.cloud.com]
4. Next to **Assertion Consumer Service** enter the **Assertion Consumer Serivce URL** found in Citrix Cloud XML file (step 8 above)Example:  [https://saml.cloud.com/saml/acs]
5. **NameID format** should stay as the default: urn.oasis:names:tc:SAML:1.1:nameid-format:emailAddress
6. Next to **NameID attribute** input the attribute that maps to your email address. If possible, I always recommend choosing Duo's preconfigured attributes, in this case This will allow you to change Duo SSO Authentication Source in the future, if needed. For example, from AD to a SAML IdP.
7. Next to **Signing options** leave both **Sign response** and **Sign assertion** checked.
8. Set the following attribute values in the **Mapp Attributes** section:
   1. IdP Attribute = < Display Name > & SAML Response Attribute = DisplayName
   2. IdP Attribute = < Email Address > & SAML Response Attribute = cip_email
   3. IdP Attribute = < First Name > & SAML Response Attribute = FirstName
   4. IdP Attribute = < Last Name > & SAML Response Attribute = LastName
   5. IdP Attribute = ObjectSID & SAML Response Attribute = cip_sid
   6. IdP Attribute = userPrincipalName & SAML Response Attribute = cip_upn
   7. IdP Attribute = ObjectGUID & SAML Response Attribute = cip_oid
9. Scroll down to the **Policy** section and choose the policy you wish to implement for this application.
10. Scroll down to the **Settings** section and next to **Name** add Citrix Workspace. You may also want to configure other options under this section, depending on how you have Duo MFA configured for your users.
11. Scroll to the bottom and click **Save**

#### Back in your Citrix Cloud account
1. Scroll down and click **Test and Finish**
2. In the **Citrix Cloud Menu**, navigate to **Workspace Configuration** and click on the **Authentication** tab.
3. Change the **Workspace Authentication** option to **SAML 2.0**. A dialog will appear warning you about your change.
4. Check the **I understand the impact** box then click **Confirm**.
   
#### Adding your Citrix Cloud application to Duo Central
1. In **Citrix Cloud administration** UI, click the **hamburger menu** and select the **Workspace Configuration** tab.
2. Copy the **Workspace URL**.
3. Return to the **Duo Admin panel** and navigate to **Single Sign-on** - Duo **Central**
4. Click the **+ Add Tile** button and select add a **Bookmark tile**.
5. In the **Name field**, type a name such as **Citrix Workspace**.
6. **Paste** the copied **Workspace URL** into the URL field.
7. Alter any of the additional fields as needed, such as an image and click **Save**.
   
You are now ready to test SSO authentication into Citrix Cloud (end user experience is called Citrix Workspace)! To test either go to Duo Central and click on the tile or navigate directly to your Citrix Workspace URL.


   


