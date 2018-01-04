# OAuth Basics

Fabrikam, Inc. has been using a number of security related protocols over the years for their solutions, but since the Drone Delivery Service is a new service based on current architectural principles they do not want to use legacy protocols if iot can be avoided. Based on this they have decided to implement OAuth and OpenID Connect for this implementation. There might be cases where they need to support SAML for legacy integrations, so the components will not necessarily be "locked down" to not accept SAML in any form, but if this type of integration is necessary it will be handled by modules outside the core components. (More on this in B2C and API Management sections.)

As Fabrikam are using Azure Active Directory the support they need for OAuth and OpenID Connect is built into the Identity Access Management platform they have chosen. The whole purpose of Azure AD is authenticating users and apps, and authorizing access to resources. AuthN and AuthZ could be said to be the high level goal. OAuth (and OpenID Connect) are the low level parts of how it actually happens.

> Authentication (AuthN) is about answering the question "who are you?". Authorization (AuthZ) follows, and is about answering the question "should you be allowed to access this resource?".

## OAuth vs OpenID Connect
This is a point that causes some confusion so a few lines explaining the differences are in order. When OAuth came about it addressed some specific use cases, but was not designed (nor intended) to solve all authentication scenarios. The scenario where you use an external Identity Provider (IdP) to have users interactively sign in to your application wasn't really covered by the OAuth standard. This led to different approaches being implemented that all had their own nuances. If one wanted to sign in with Google accounts instead of Microsoft accounts (on basis of both supporting OAuth) a couple of lines of code would need to be adapted, and if you needed to support both at the same time extra code would need to be added. The implementations were mostly similar, but not identical. This eventually led to a standardization effort to actually make this work without rewriting code when supporting multiple Identity Providers.

OpenID Connect builds on top of OAuth though, and is not meant to replace it. For a non-interactive server to server use case it's still perfectly valid to not implement OpenID Connect.

> For an interactive login by an end-user OpenID Connect is usually what you want. For a server type non-interactive login OAuth is what you want.

Fabrikam, Inc. will be supporting both interactive and non-interactive logins in their service, so they will need to support both protocols.

## App types
Developers are used to thinking about tiers and layers. The component driving the user interface is separated from the component driving the SQL database in the background. Authentication works in a similar way. The "Login" button visible to a user does not trigger the same code as a server side process needing to look up all users in a given office location. The verbiage of the protocols do not use this term, but the different microservices and modules in the Drone Delivery Service can be said to be different "app types" as a means for describing these different ways of driving the auth.

_Native app_  
With native app we mean something the end-user installs. This could be an app for an iPhone or an Android mobile device, but also a Win32 client for a Windows box. This is considered an untrusted client.

_Web app_  
This is the classic "signing in on a web page" scenario. There is no client installed on the end-users device (downloaded JavaScript code does not count here), and execution is controlled by a server. This is usually considered a trusted client.

_Service/daemon app_  
This is an odd case where there is a client installed, and it authenticates as a user although not necessarily having an interactive element. Think a service running in the background doing things, either as a server side component, or part of the client install on the end user device. This could be considered both a trusted and an untrusted client depending on the specific implementation scenario.

> If an end user has direct access to the executable the client is considered untrusted - aka unable to keep secrets.

_Web APIs_  
What about APIs, aren't they a type of app as well? They are an important part of the picture yes, and it is of course very common to build applications where most of the functionality is provided by an API having only a thin piece of user interface on top exposing things to a user. But keep in mind that the API is usually not the component doing the login. The API mainly receives a token and verifies whether these are valid keys to the kingdom. The UI should contain the code needed to acquire said token.

_Single-Page Apps (SPAs)_  
Fair enough, but what about Single-Page Apps running some JavaScript framework? It's sort of trusted since it runs on a server, but at the same time most of the code is downloaded to the client so wouldn't this potentially imply placing trusted bits in an untrusted place? Yes, this is a tricky one. There are different ways to handle this depending on your chosen frameworks. 

## OAuth Flows
In OAuth parlance the different mechanisms for logging in are called "flows". Why not app types or something similar? Well, while there are natural ways to map the mechanisms to different type of apps there is nothing technically preventing you from doing things that aren't necessarily recommended. The OAuth backend does not care about this, and does not force you into doing things a certain way, so a more generic term is more suitable.

These are only some of the available OAuth flows - Fabrikam, Inc. might need more advanced flows for some of their modules, but these are the most basic ones.

_Client credentials_  
This flow is very similar to the traditional username/password login mechanism. You have a clientId, and a clientSecret which for most purposes behave similarly except the secret is usually something you would not be able to memorize at first try. These credentials are not intended for being used by actual end users however, and rather identifies the app itself. While you can have multiple secrets per app, there is only the one id, and it is in no way scalable to provide unique identities per actual end user of an app. This flow is intended for trusted clients, and you should never ever store the secret outside a server context! The secrets can be changed, and rolled over on an interval, but updating this in a bunch of installations is a major hassle.

Use this flow for server type apps. Client Credentials logically maps to the Web App in the Azure portals.

_Authorization code_  
This is a very common use case, and unfortunately the one that is hardest to grasp at first. The client "signals" that it would like to sign in, and identifies itself with the clientId of the app. This in turn triggers a redirect to a web view hosted by Azure AD where the user types in their username and password, and then the user is sent back to the app with a token. The important part here is that the app never has access to the actual credentials of the user, while at the same time the authentication process is tied to the app so you can't randomly invoke this process from a third-party app. 

Use this flow for end-user interactive apps. Authorization Code logically maps to the Native App in the Azure portals.

_Password grant_  
This flow is used when you need to identify as a user, and not an app, but want to do it in a non-interactive way. Think something like the user typing in credentials during install, and then have the app use this in the background for subsequent logins.
This flow is suitable for service/daemon type of apps. It doesn't map directly to an app type in the Azure portals, but there are restrictions enforced in the background so it might not work for all use cases.

_Combo apps_  
There are scenarios where you possibly need both a web page and a native app. There's nothing preventing you from registering multiple apps, and using several clientIds. Maybe you want a different app for iOS and Android - go ahead. Maybe you have a web app where you don't need to use client credentials. You can mix and match these things as you like - there is no one-to-one mapping between app types and flows so you can be flexible.

> Pro tip: Follow these general designations, and think twice before implementing "clever" variants of these in your own implementations.

A common "clever implementation" worth highlighting is abuse of the password grant flow. Frequently there is a request to be able to modify the user experience for signing in; either because one doesn't like the layout of the default Microsoft-branded look, or reducing the number of steps involved. And then you're thinking "hey, if I go with the password grant I can have the user type in their credentials in a UI I control 100%, and do the actual authentication in the background". Yes, technically you can do this. And you should not do it... The whole point of driving the user to Azure AD is the security aspect of you not taking care of this in your app. If you implement this by handling the user's credentials in clear text you introduce a whole new bundle of security issues. 

Does this mean it is never ok to provide a customized login experience? Well, there are different levels of customization of the Azure AD login. You can change the branding and texts quite easily, and with Azure AD B2C work is being made to provide richer customization. There could be times when the password grant mechanism can be "acceptable" - for instance in a non-public app installed only on approved corporate computers for internal users. Fabrikam, Inc. want to appear trustworthy with their service, and will avoid these workarounds.