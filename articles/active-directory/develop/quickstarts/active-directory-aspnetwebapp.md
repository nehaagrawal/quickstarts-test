---
title: Azure AD v2 ASP.NET web server Quickstart | Microsoft Docs
description: Implementing Microsoft Sign-In on an ASP.NET solution with a traditional web browser based application using OpenID Connect standard
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''

ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/10/2018
ms.author: andret
ms.custom: aaddev 

---

# Add sign-in with Microsoft to an ASP.NET web app

This quickstart contains a code sample that demonstrates how an ASP.NET Web App can sign in personal (hotmail.com, live.com, others) and work and school accounts from any Azure Active Directory instance.

![How the sample app generated by this Quickstart works](media/active-directory-aspnetwebapp/aspnetbrowsergeneral.png)

![How the sample app generated by this Quickstart works](media/active-directory-aspnetwebapp/aspnetwebapp-intro.png)

> [!div renderon="docs"]
> ## Step 1: Register your application in Azure portal
> 
> 1. To register an application, go to the [Azure AD - Application Registration](https://portal.azure.com/signin/index/?Microsoft_AAD_RegisteredApps=true#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs)
> 1. Enter a name for your application and click **Create**
> 1. Follow the steps to add *https:<span/>//localhost:44368/* as redirect URL

> [!div class="sxs-lookup" renderon="portal"]
> ## Step 1: Configure your application
> For the code sample for this quickstart to work, you need to add a reply URL to *https:<span/>//localhost:44368/* as redirect URL and enable *Implict flow*
> > [!div id="makechanges" class="hidden"]
> > [Make these changes for me]()
>
> > [!div id="appconfigured" class="hidden"]
> > ![Already configured](media/active-directory-windesktop/checkmark.png) Your application is configured with these attributes

## Step 2: Download your web server or project

- [Download the Visual Studio 2017 project](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-DotNet/archive/master.zip)

## Step 3: Configure your Visual Studio project

1. Open the project in Visual Studio
1. Edit **Web.config** and replace `Enter_the_Application_Id_here` with the Application ID from the application you just registered:

    ```xml
    <add key="ClientId" value="Enter_the_Application_Id_here" />
    ```

## More information

### OWIN middleware NuGet packages

You can setup the authentication pipeline with cookie-based authentication using OpenId Connect in ASP.NET with Owin Middleware packages. You can install thse packages by running the following commands in Visual Studio's *Package Manager Console*::

```powershell
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.SystemWeb
```

### Owin Startup Class

OWIN Middleware uses a Startup class that is executed when the hosting process initializes. Below shows the parameter used by this Quickstart:

```csharp
public void Configuration(IAppBuilder app)
{
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

    app.UseCookieAuthentication(new CookieAuthenticationOptions());
    app.UseOpenIdConnectAuthentication(
        new OpenIdConnectAuthenticationOptions
        {
            // Sets the ClientId, authority, RedirectUri as obtained from web.config
            ClientId = clientId,
            Authority = authority,
            RedirectUri = redirectUri,
            // PostLogoutRedirectUri is the page that users will be redirected to after sign-out. In this case, it is using the home page
            PostLogoutRedirectUri = redirectUri,
            Scope = OpenIdConnectScope.OpenIdProfile,
            // ResponseType is set to request the id_token - which contains basic information about the signed-in user
            ResponseType = OpenIdConnectResponseType.IdToken,
            // ValidateIssuer set to false to allow personal and work accounts from any organization to sign in to your application
            // To only allow users from a single organizations, set ValidateIssuer to true and 'tenant' setting in web.config to the tenant name
            // To allow users from only a list of specific organizations, set ValidateIssuer to true and use ValidIssuers parameter 
            TokenValidationParameters = new TokenValidationParameters()
            {
                ValidateIssuer = false
            },
            // OpenIdConnectAuthenticationNotifications configures OWIN to send notification of failed authentications to OnAuthenticationFailed method
            Notifications = new OpenIdConnectAuthenticationNotifications
            {
                AuthenticationFailed = OnAuthenticationFailed
            }
        }
    );
}
```

> |Where  |  |
> |---------|---------|
> |ClientId     |Application Id from the application registered in the Azure Portal|
> |Authority | The STS endpoint fo user to authenticate. Usually https://login.microsoftonline.com/{tenant}/v2.0 for public cloud, where {tenant} is the name of your tenant, your tenant Id, or *common* for a reference to the common endpoint (used for multi-tenant applications)|
> |RedirectUri     |URL where users are sent after authentication against Azure AD v2 Endpoint|
> |PostLogoutRedirectUri     |URL where users are sent after signing-off|
> |Scope     |The list of scopes being requested, separated by spaces|
> |ResponseType     |Request a respose contains an Id Token|
> |TokenValidationParameters     | A list of parameters for token validation. In this case, `ValidateIssuer` is set to `false` to indicate that it can accept sign-ins from any personal, or work or school accounts|
> |Notifications     | A list of delegates that can be executed on different *OpenIdConnect* messages|

### Initiate an authentication challenge

You can force user to sign in by requesting an authentication challenge in your controller:

```csharp
public void SignIn()
{
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(
            new AuthenticationProperties{ RedirectUri = "/" },
            OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}
```

> [!TIP]
> Requesting an authentication challenge using the method above is optional and commonsly used when you want a view to be available for both authenticated and non-authenticated users. You can protect controllers by using the method described in the next section.

### Protect a controller or a controller's method

You can protect a controller or controller methodss using the `[Authorize]` attribute. This attribute restricts access to the controller or methods by only allowing authenticated users - which means that authentication challenge can be started to access the controller if user is not authenticated.

## What is next

Try out the ASP.NET tutorial for a complete step-by-step guide on building applications and building new features, including a full explanation of this Quickstart:

### Learn the steps to create the application used in this Quickstart

> [!div class="nextstepaction"]
> [Sign-in Tutorial](https://docs.microsoft.com/azure/active-directory/develop/guidedsetups/active-directory-aspnetwebapp)

[!INCLUDE [Help and support](../../../../includes/active-directory-develop-help-support-include.md)]