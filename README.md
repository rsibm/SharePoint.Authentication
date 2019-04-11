# SharePoint.Authentication

SharePoint.Authentication is an inject-able SharePoint context and token helper which can be used in multi-tenant applications.

Reason I came up with this project is due to problems I've met while creating high trust multi-tenant application. In the project I've worked on had different client id and secret for each tenant. As you may recall, SharePoint context provider automatically added to web project currently rely on only one client id and secret which must be added to web.config file. This was not the solution I wanted because, application had different client id and secret (provided by seller dashboard) for low trust part of the app and had different client id and secret for each tenant for high trust.

To authentication layer to function properly, I wanted it to instantiated per tenant/user and wanted it to inject-able via Unity container. This is the solution I came up with to fix that issue.

## DISCLAIMER

This library is just an extension for existing code provided by Microsoft. Almost all code here is copied from Microsoft provided authentication and context provider code, I made few changes to work with scenarios described below, but all props goes to engineers who wrote it from scratch.

## Getting started

You must implement few interfaces and abstract classes in order to use this in an application.

### IAuthenticationParameters

This is the base interface which used to define parameters needed for SharePoint authentication. You do not have to implement this interface directly, but implement two abstract classes implemented from this interface.

```csharp
namespace SharePoint.Authentication
{
    public interface IAuthenticationParameters
    {
        string ClientId { get; }
        string ClientSecret { get; }
        string IssuerId { get; }
        string HostedAppHostNameOverride { get; }
        string HostedAppHostName { get; }
        string SecondaryClientSecret { get; }
        string Realm { get; }
        string ServiceNamespace { get; }

        string SigningCertificatePath { get; }
        string SigningCertificatePassword { get; }
        X509Certificate2 Certificate { get; }
        X509SigningCredentials SigningCredentials { get; }
    }
}
```

It's not required to implement all members of this interface, you can implement members only used in your application. For example, if you are using ClientId and ClientSecret for authentication, you can add implementation only for those, you can leave others empty.

### AcsAuthenticationParameters

This abstract class is implemented from ```IAuthenticationParameters```. If you need to use ```AcsTokenHelper```, ```SharePointAcsContext``` or ```SharePointAcsContextProvider``` inside your application, you have to implement this class to provide required parameters.

```csharp
public class SampleAcsAuthenticationParameters : AcsAuthenticationParameters
{
    public override string ClientId { get; }
    public override string IssuerId => null;
    public override string HostedAppHostNameOverride => null;
    public override string HostedAppHostName => null;
    public override string ClientSecret { get; }
    public override string SecondaryClientSecret => null;
    public override string Realm => null;
    public override string ServiceNamespace => null;
    public override string SigningCertificatePath => null;
    public override string SigningCertificatePassword => null;
    public override X509Certificate2 Certificate => null;
    public override X509SigningCredentials SigningCredentials => null;
    public SampleAcsAuthenticationParameters()
    {
        ClientId = ConfigurationManager.AppSettings["sampleMvc:AcsClientId"];
        ClientSecret = ConfigurationManager.AppSettings["sampleMvc:AcsClientSecret"];
    }
}
```

As you can see in sample above, here I make sure that client id and client secret is loaded once from constructor of parameters class. When you use dependency injection to inject instance of this class, you can make it a singleton to improve performance.

### HighTrustAuthenticationParameters

This is again an abstract class implemented from ```IAuthenticationParameters```. You can use implementation of this class to use ```HighTrustTokenHelper```, ```SharePointHighTrustContext``` or ```SharePointHighTrustContextProvider``` inside your application.
