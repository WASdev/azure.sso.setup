# azure.sso.setup
Set up Microsoft :registered: Entra:tm: ID or Azure:tm: AD as an SSO identity provider for WebSphere:tm: Application Server and Liberty clients

## Entra ID setup examples 
- [Setting up Microsoft Entra ID as an OIDC OP for WebSphere Application Server and Liberty clients](entraOidc.md)
  - The steps in [Setting up Azure as an OIDC OP for WebSphere Application Server and Liberty clients](https://github.com/WASdev/azure.sso.setup/blob/main/azureOidc.md) on this site have been modified to account for using the Azure portal or Entra ID admin center.  For steps to set up Entra ID as an OIDC OP, see [Setting up Azure as an OIDC OP for WebSphere Application Server and Liberty clients](https://github.com/WASdev/azure.sso.setup/blob/main/azureOidc.md).

## Azure AD SSO setup examples 
- [Setting up Azure as an OIDC OP for WebSphere Application Server and Liberty clients](azureOidc.md)
  - This example is a fast path to get you started with using [Microsoft :registered: Entra:tm: ID (formerly Azure:tm: AD)](https://azure.microsoft.com/services/active-directory/) as your OpenID Connect provider (OP) for the WebSphere Application Server traditional and Liberty OIDC relying parties (RP)
- [Modifying an Azure OIDC OP for WebSphere Application Server and Liberty clients](azureOidcMod.md)
  - This example shows how to modify an existing Azure app registration to add a WebSphere traditional or Liberty OIDC relying parties (RP).

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License
Unless otherwise noted in a script:<br/>
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
