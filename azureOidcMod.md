# Modify Azure OIDC for WebSphere and Liberty clients
---

# Description

Modifying a Microsoft Azure OIDC OP for WebSphere Application Server and Liberty clients.  This example shows how to modify an existing Microsoft :registered: Azure:tm: app registration to add a WebSphere:tm: traditional or Liberty OIDC relying parties (RP).  The steps are similar for the Entra:tm: ID console.

# References

Here are some links from Microsoft that contain more detailed configuration information:
- [Quickstart: Set up a tenant](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-create-new-tenant)
- [Quickstart: Register an application with the Microsoft identity platform](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure an application to expose a web API](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-configure-app-expose-web-apis)
- [Quickstart: Configure a client application to access a web API](https://learn.microsoft.com/en-gb/azure/active-directory/develop/quickstart-configure-app-access-web-apis)


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

   
# Procedure

## Login to the Azure portal

1. <font size="+1">Login to the  <a href="https://portal.azure.com">Azure portal</a>  at <a href="https://portal.azure.com">https://portal.azure.com</a>.</font>


1. <font size="+1">If you have access to multiple tenants, perform the following actions to choose the tenant in which you want to register the application:</font>
   - Click&nbsp;&nbsp;![](files/AzFilter.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu to access the <b>Directories + subscriptions filter</b> menu.
   - Switch to the tenant in which you want to register the application.
   - After you select your tenant, click&nbsp;&nbsp;![](files/AzButton.png?preventCache=1551478986083)&nbsp;&nbsp;in the top menu on the left to return to the **Azure services** menu.


## Navigate to your App Registration

   ![](files/AzSearch.png?preventCache=1551478986083)

1. <font size="+1">In the search box in the menu bar at the top, search for <b>Azure Active Directory</b> then click <b>Azure Active Directory</b></font>

	![](files/AzADChoose.png?preventCache=1551478986083)

1. <font size="+1">Under <b>Manage</b> in the menu on the left, click <b>App Regsistrations</b></font>

	![](files/AzApps.png?preventCache=1551478986083)

1. Click the application that you want to use.

## Get the client ID and discovery URL values
Note values to use when when configuring WebSphere or Liberty later in this task.
  - The values for the **client ID** and **tenant ID** 
    - All your client secrets use the same client ID.
   ![](files/AzUpdateApp.png?preventCache=1551478986083)
  - Your discovery URL is **https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration**
    - where **{tenant}** is the name of your tenant.

## Create a client secret

1. <font size="+1">Click the link next to <b>Client credentials</b></font>
1. <font size="+1">Click <b>New client secret</b></font>
       ![](files/AzExistingSecret.png?preventCache=1551478986083)
1. <font size="+1">Enter a description for your new secret and the expiration, then click <b>Add</b></font>
       ![](files/AzNewSecret.png?preventCache=1551478986083)
1. <font size="+1">![](files/AzCaution.gif?preventCache=1551478986083) <b>Caution</b>: Be sure to note the value that is generated for the <font size="+2"><b>client secret</b></font> to use when configuring WebSphere or Liberty later in this task.  You might not see this value again later.</font>

   ![](files/AzWasSecret.png?preventCache=1551478986083)


## Add a redirect URI:
<font size="+1">Under <b>Manage</b>, click <b>Authentication</b>
- <font size="+1">![](files/AzRedArrow.gif?preventCache=1551478986083) If the **Web** platform already exists:</font>
  1. Click **Add URI**
    ![](files/AzAddUri.png?preventCache=1551478986083)
  1. Enter your <b>Redirect URI</b>: <i>https://(hostname):(port)/(contextRoot)/(identifier)</i>
     - If you have your redirect URI, enter it now.  
     - Otherwise, see the **Before you begin** section for how to determine your redirect URI.
- <font size="+1">![](files/AzRedArrow.gif?preventCache=1551478986083) Otherwise, if the Web platform does not exist:</font> 
  1. Click <b>Add a platform</b>
     ![](files/AzAddPlatform.png?preventCache=1551478986083)
  1. Click **Web**
  1. <font size="+1">Fill in the information on the <b>Configure Web</b> panel:</font>
     ![](files/AzAddRedirect.png?preventCache=1551478986083)
     1. <b>Redirect URI</b>: <i>https://(hostname):(port)/(contextRoot)/(identifier)</i>
        - If you have your redirect URI, enter it now. 
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

