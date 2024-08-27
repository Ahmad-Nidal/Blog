



```md
notes:
this article was written before understanding what OpenIddict is exactly and why Microsoft provides its package `Microsoft.AspNetCore.Authentication.OpenIdConnect` for OpenIdConnect. The article is a bit misleading and needs to be updated. 
```


---



# OpenID Connect Authentication Client With OpenIddict

## What is OpenIddict?

**OpenIddict** is a comprehensive framework designed to provide flexible support for implementing OpenID Connect clients, servers, and token validation within any ASP.NET Core 2.1 (or higher) application, as well as ASP.NET 4.6.1 (and higher) applications.

Unlike many other identity providers, OpenIddict is not a ready-to-use, out-of-the-box solution. Instead, it acts as a framework that requires custom code to become fully operational—typically, this involves implementing an authorization controller.

---

*Source: [OpenIddict GitHub Repository](https://github.com/openiddict/openiddict-core)*

## Common Turnkey Solutions for OpenID Connect

If you're looking for ready-made solutions for implementing OpenID Connect, the following options are popular choices you can look into:

- **Orchard Core OpenID Module**  
  Documentation: [Orchard Core OpenID](https://docs.orchardcore.net/en/latest/reference/modules/OpenId/)  
  Source Code: [Orchard Core OpenID GitHub](https://github.com/OrchardCMS/OrchardCore/blob/main/src/OrchardCore.Modules/OrchardCore.OpenId/)

- **ABP Framework OpenIddict Module**  
  Documentation: [ABP OpenIddict Module](https://abp.io/docs/latest/modules/openiddict)  
  Source Code: [ABP OpenIddict GitHub](https://github.com/abpframework/abp/blob/dev/modules/openiddict/src/Volo.Abp.OpenIddict.AspNetCore/Volo/Abp/OpenIddict/)

Additionally, OpenIddict offers a clean and straightforward example called **Velusia**:  
[Velusia Sample](https://github.com/openiddict/openiddict-samples/tree/dev/samples/Velusia)

## Configuring an OpenID Connect Client

To set up an OpenID Connect client, you can utilize either the `AddOpenIdConnect` or `AddOpenIddict` extension methods in your ASP.NET Core project.

### Example Configurations:

**1. Using `AddOpenIdConnect`:**

```csharp
// Example configuration using AddOpenIdConnect
```

**2. Using `AddOpenIddict`:**

```csharp
// Example configuration using AddOpenIddict
```

It's crucial to correctly set the callback path (RedirectUri) and to implement an authentication controller that handles the callback. The authentication controller should map the user claims, and sign in the user once they have been authorized by the identity provider.

### Authentication Controller Examples

Here are some examples of authentication controllers:

- **ABP Framework:** [AuthorizeController.cs](https://github.com/abpframework/abp/blob/dev/modules/openiddict/src/Volo.Abp.OpenIddict.AspNetCore/Volo/Abp/OpenIddict/Controllers/AuthorizeController.cs)
- **Orchard Core:** [AccessController.cs](https://github.com/OrchardCMS/OrchardCore/blob/8380e35cb1086e0a33103a606faeb3ba6c8b83f8/src/OrchardCore.Modules/OrchardCore.OpenId/Controllers/AccessController.cs#L184)
- **Velusia Sample:** [AuthenticationController.cs](https://github.com/openiddict/openiddict-samples/blob/dev/samples/Velusia/Velusia.Client/Controllers/AuthenticationController.cs)

### LocalUser vs ExternalUser

- **LocalUser:** A user stored within the application's database.
- **ExternalUser:** A user managed by an external identity provider.

When an external identity provider is used for authentication, the application should map the external user to a local user account. Typically, the application searches for a local user with the same email address as the external user. If a match is found, the external user is mapped to that local user. If no match is found, a new local user is created, and the external user is mapped to this newly created account.

## Velusia Sample Overview

The Velusia sample demonstrates the authorization code flow using an ASP.NET Core application acting as the client.

The sample comprises two projects:

- **Velusia.Client**
- **Velusia.Server**

To run the sample:

1. Clone the samples repository: [OpenIddict Samples GitHub](https://github.com/openiddict/openiddict-samples).
2. Run the `Velusia.Server` project located in `samples/Velusia/`.
3. Run the `Velusia.Client` project in `samples/Velusia/`.

If you're using Visual Studio, you can configure multiple startup projects to run both the client and server simultaneously. Additionally, you can scope the Visual Studio Solution Explorer to show only the Velusia folder.

## Helpful Links

For more in-depth information and resources, check out the following links:

- [Introducing the OpenIddict Client](https://kevinchalet.com/2022/02/25/introducing-the-openiddict-client/)
- [Getting Started with the OpenIddict Web Providers](https://kevinchalet.com/2022/12/16/getting-started-with-the-openiddict-web-providers/)
- [OpenIddict Documentation](https://documentation.openiddict.com/)
- [The OpenID Connect Handbook](https://auth0.com/blog/the-openid-connect-handbook)
