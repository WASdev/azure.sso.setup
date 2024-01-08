# Setting up Azure as an OIDC OP for WebSphere Application Server and Liberty clients
---

# Description

This document is indended as a fast path to get you started with using [Microsoft :registered: Entra:tm: ID (formerly Azure:tm: AD)](https://azure.microsoft.com/services/active-directory/) as your OpenID Connect provider (OP) for the WebSphere:tm: Application Server traditional and Liberty OIDC relying parties (RP).  

# References

Here are some links from Microsoft that contain more detailed configuration information:
- [Quickstart: Set up a tenant](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-create-new-tenant)
- [Quickstart: Register an application with the Microsoft identity platform](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure an application to expose a web API](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-configure-app-expose-web-apis)
- [Quickstart: Configure a client application to access a web API](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-configure-app-access-web-apis)

Here is a link to an Entra ID setup that might be is less detailed, but includes using either Power Pages and Azure:
- [Set up an OpenID Connect provider with Microsoft Entra ID](https://learn.microsoft.com/en-us/power-pages/security/authentication/openid-settings)

# Background

The Azure AD application configuration and OIDC RP configurations work in concert with each other.  The Azure config requires a <i>redirect URL</i> from the RP.  The RP configuration requires the <i>client ID</i>, <i>client secret</i>, and <i>discovery URL</i> from the Azure configuration.  Whichever configuration you choose to do first, you must go back and edit that configuration using information from the second.  For instance, if you configure Azure first, after configuring the RP, you go back into your Azure config and add the <i>redirect URL</i>.  

If the RP (WebSphere traditional or Liberty) and Azure administration roles are separated in your organization, it is best to perform the RP configuration first, then provide the <i>redirect URL</i> to your Azure administrator.  The Azure administrator then returns the <i>client ID</i>, <i>client secret</i>, and <i>discovery URL</i> to you.

# Before you begin

## Configure your OIDC RP: 

  - For WebSphere Application Server Traditional, see [Configuring an OpenID Connect Relying Party](https://www.ibm.com/docs/en/was-nd/9.0.5?topic=SSAW57_9.0.5/com.ibm.websphere.nd.multiplatform.doc/ae/tsec_oidconfigure.html) and [OpenID Connect Relying Party custom properties](https://www.ibm.com/docs/en/was-nd/9.0.5?topic=SSAW57_9.0.5/com.ibm.websphere.nd.multiplatform.doc/ae/csec_oidprop.html).
    - On the _Import the OpenID connect provider's SSL signer certificate to the WebSphere Application Server truststore_ step, use the following data:
      - host: **login.microsoftonline.com**
      - port: **443**
  - For Liberty, see [Configuring an OpenID Connect Client in Liberty](https://www.ibm.com/docs/en/was-liberty/base?topic=SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_config_oidc_rp.html)
    - On the step to _Configure the truststore of the server to include the signer certificates of the OpenID Connect Providers that are supported_ using the [Adding trusted certificates in Liberty](https://www.ibm.com/docs/en/was-liberty/base?topic=SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_add_trust_cert.html) topic in IBMDOCS, the signer certificate that you want is for the following endpoint:
      - https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
      - Where **{tenant}** is the name of your tenant.

  - The <b>Redirect URI</b> that you will use for the RP when configuring Azure is <b><i>https://(hostname):(port)/(contextRoot)/(identifier)</i></b>, where:
    - (hostname):(port): 
      - The hostname and SSL port of the WebSphere or Liberty server.
    - (contextRoot):
      - Liberty : 
        - Replace the value with **oidcclient/redirect**
      - WebSphere traditional: 
        - The default value is **oidcclient**
        - This is the context root of **WebsphereOIDCRP ear**
        - To find the value, in the Administrative console, navigate to **All Applications > WebsphereOIDCRP > Context Root for Web Modules**
          - If you installed the OIDC ear using [deployOidc.py](https://github.com/WASdev/sample.wsadmin.websphere-traditional) for use with the admin console, then you want to look for **WebsphereOIDCRP_Admin** instead of **WebsphereOIDCRP**
    - (identifier)
      - Liberty: the value for the **id** attribute of your **openidConnectClient** configuration.
      - WebSphere traditional: the value for the **provider_(id).identifier** OIDC TAI custom property.
    - **Examples** :  
      - Liberty: https://test.co:9443/oidcclient/redirect/RP1
      - Websphere traditional: https://test.co:9443/oidcclient/RP1

   


## Create an Azure AD user account

 - If you are working with a free Azure account and have not yet added any users, follow the steps to [Create a user account](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users#create-a-user-account) on [Quickstart: Create and assign a user account](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users).
     - ![](files/AzCaution.gif?preventCache=1551478986083) <b>Caution</b>: You won't be able to add a group as shown in the **Assign a user account to an enterprise application** section, so don't try it. 


# Procedure

## Login to your identity provider portal

<font size="+1">Do one of the following:</font>
- <font size="+1">Login to the <b><font size="+2">Entra console</font></b></font>
  1. <font size="+1">Login to the <a href="https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EntraNav.ReactView">Entra console</a> at <a href="https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EntraNav.ReactView">https://entra.microsoft.com/#view/Microsoft_AAD_IAM/EntraNav.ReactView</a></font>.
  2. <font size="+1">If you have access to multiple tenants, perform the following actions to choose the tenant in which you want to register the application:</font>
     1. <font size="+1">Click the settings icon&nbsp;&nbsp;![](files/gear.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu to access the <b>Directories + subscriptions filter</b> menu.</font>
     1. <font size="+1">Switch to the tenant in which you want to register the application.</font>
     1. <font size="+1">After you select your tenant, click&nbsp;&nbsp;![](files/Entra.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu on the left to return to the **Entra admin center** menu.</font>
     2. <font size="+1">Under <b>Identity</b> in the menu on the left, click <b>Applications</b>, then <b>App registrations</b></font>

- <font size="+1">Login to the <b><font size="+2">Azure portal</font></b></font>
  1. <font size="+1">Login to the <a href="https://portal.azure.com">Azure portal</a> at <a href="https://portal.azure.com">https://portal.azure.com</a></font>.
  2. <font size="+1">If you have access to multiple tenants, perform the following actions to choose the tenant in which you want to register the application:</font>
     1. <font size="+1">Click the settings icon&nbsp;&nbsp;![](files/gear.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu to access the <b>Directories + subscriptions filter</b> menu.</font>
     2. <font size="+1">Switch to the tenant in which you want to register the application.</font>
     3. <font size="+1">After you select your tenant, click&nbsp;&nbsp;![](files/AzButton.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu on the left to return to the **Azure services** menu.</font>
     1. <font size="+1">In the search box in the menu bar at the top, search for <b>Azure Active Directory</b> then click <b>Microsoft Entra ID</b></font>

     	![](files/EntraID.png?preventCache=1551478986083)

     1. <font size="+1">Under <b>Manage</b> in the menu on the left, click <b>App Regsistrations</b></font>

        ![](files/AzAppRegistrations.png?preventCache=1551478986083)
     	


## Register your application

1. <font size="+1">In the action bar, click <b>New registration</b></font>

	![](files/AzNewRegistration.png?preventCache=1551478986083)

1. <font size="+1">On the <b>Register an application</b> panel, provide the details for the application you are registering:</font>
	![](files/AzRegister.png?preventCache=1551478986083)
   - **Name**: The name of your application
   - **Supported account type**: Multitenant
	 - If you need your application to use Oauth V2.0 endpoints, select Accounts in any organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts
   - The **Redirect URI** field is optional on this page; instructions are provided later in this document for setting the Redirect URI
     - If you enter it now, be sure to select Web as the platform
1. <font size="+1">Click <b>Register</b></font>
   - Note values to use when when configuring WebSphere or Liberty later in this task.
     - Your discovery URL is **https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration**
       - where **{tenant}** is the name of your tenant.
       - you can see this URL if you click the **Endpoints** link at the top of the **App registrations page.  It is in the the **OpenID Connect metadata document** field.
     - The generated for the **client ID** and **tenant ID** 
       - All your client secrets use the same client ID.
   ![](files/AzNewApp.png?preventCache=1551478986083)
1. <font size="+1">Create a client secret</font>
   1. <font size="+1">Next to <b>Client credentials</b>, click <b>Add a certificate or secret</b></font>
   1. <font size="+1">Click <b>New client secret</b></font>

	  ![](files/AzClientSecret.png?preventCache=1551478986083)

   1. <font size="+1">Enter a description for your new secret and the expiration, then click <b>Add</b></font>

	  ![](files/AzSecret.png?preventCache=1551478986083)

   1. <font size="+1">![](files/AzCaution.gif?preventCache=1551478986083) <b>Caution</b>: Be sure to note the value that is generated for the <font size="+2"><b>client secret</b></font> to use when configuring WebSphere or Liberty later in this task.  You cannot view this value again later.</font>

      ![](files/AzTestSecret.png?preventCache=1551478986083)

## Add permissions

1. <font size="+1">Under <b>Manage</b>, click <b>API Permissions</b>, then <b>Add a permission</b></font>

   ![](files/AzPermissions.png?preventCache=1551478986083)


1. <font size="+1">Click <b>Microsoft Graph</b></font> 

   ![](files/AzRequestPermission.png?preventCache=1551478986083)

1. <font size="+1">Click <b>Delegated permissions</b></font>

   ![](files/AzGraphPermissions.png?preventCache=1551478986083)

1. <font size="+1">Check the <b>Openid permissions</b>, then click <b>Add permissions</b>:</font>
   - openid
   - profile
   - (Optional) Check any other permissions that your application might require.

   ![](files/AzDelegatedPermissions.png?preventCache=1551478986083)


1. <font size="+1">Click <b>Grant admin consent</b>, then click <b>Yes</b></font> 

   ![](files/AzGrantAdminConsent.png?preventCache=1551478986083)

## Expose an API

1. <font size="+1">Under <b>Manage</b>, click <b>Expose an API</b> &gt; <b>Add a scope</b> &gt; <b>Save and Continue</b></font>

   ![](files/AzScope1.png?preventCache=1551478986083)

1. <font size="+1">Fill in the required fields, then click <b>Add scope</b>:</font>
   ![](files/AzScope2.png?preventCache=1551478986083)

   - **Scope name** = default
   - **Who can consent** = Admins and users
   - Admin consent display name
   - Admin consent description
   - **State** = Enabled


## Add a redirect URI:
1. <font size="+1">Under <b>Manage</b>, click <b>Authentication</b> &gt; <b>Add a platform</b> &gt; <b>Web</b></font>

   ![](files/AzAddPlatform.png?preventCache=1551478986083)


1. <font size="+1">Fill in the information on the <b>Configure Web</b> panel:</font>
   ![](files/AzAddRedirect.png?preventCache=1551478986083)
   1. <b>Redirect URI</b>: <i>https://(hostname):(port)/(contextRoot)/(identifier)</i>
      - If you have your redirect URI, enter it now.                                   a
	  - Otherwise, see the **Before you begin** section for how to determine your redirect URI.
   1. <b>Implicit grant and hybrid flows</b>:
      - Check both **Access tokens** and **ID tokens**

1. <font size="+1">Click <b>Configure</b></font>


# What to do next

1. <font size="+1">Use the <b>client ID</b>, <b>client secret</b>, and <b>discovery URL</b> to complete your OIDC configuration on WebSphere or Liberty</font>

   - For WebSphere Application Server Traditional, see [OpenID Connect Relying Party custom properties](https://www.ibm.com/docs/en/was-nd/9.0.5?topic=SSAW57_9.0.5/com.ibm.websphere.nd.multiplatform.doc/ae/csec_oidprop.html).
   - For Liberty, see [Configuring an OpenID Connect Client in Liberty](https://www.ibm.com/docs/en/was-liberty/base?topic=SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_config_oidc_rp.html).


1. <font size="+1">(<b>Optional</b>): If your RP is WebSphere traditional:</font>
   - See the [Configuring the OIDC TAI to perform RP-Initiated Logout](https://www.ibm.com/docs/en/was-nd/9.0.5?topic=SSAW57_9.0.5/com.ibm.websphere.nd.multiplatform.doc/ae/oidc_rp_initiated.html) task in IBMDOCs to determine if you want to use RP-Initiated logout.
   - If you want to perform RP-Initiated logout, perform the configuration on WebSphere, then use the <b>provider_(id).endSessionRedirectUrl</b> to complete configuration in Azure:


     1. <font size="+1">Login to the  <a href="https://portal.azure.com">Azure portal</a>  at <a href="https://portal.azure.com">https://portal.azure.com</a>.</font>

     1. <font size="+1">If you have access to multiple tenants, perform the following actions to choose the in which your application definition resides:</font>
        - Click&nbsp;&nbsp;![](files/AzFilter.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu to access the <b>Directories + subscriptions filter</b> menu.
        - Switch to the tenant in which your application definition resides.
        - After you select your tenant, click&nbsp;&nbsp;![](files/AzButton.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu on the left to return to the **Azure services** menu.
     1. <font size="+1">Click  <b>More services</b> &gt; <b>Azure Active Directory</b></font>
     1. <font size="+1">Under Manage, click  <b>App registrations</b></font>
     1. <font size="+1">Click the application that you want to update.</font>
     1. <font size="+1">Under Manage, click <b>Authentication</b>.</font>
     1. <font size="+1">In the <b>Platform configurations</b> panel, enter your <i>endSessionRedirectUrl</i> in the <b>Front-channel logout URL</b> field, then click <b>Save</b>.</font>

        ![](files/AzLogout.png?preventCache=1551478986083)

