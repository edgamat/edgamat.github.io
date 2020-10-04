---
layout: post
title: 'Integrating Angular with an Identity Provider'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Security is a fundamental aspect of any website application. This is especially true for Angular applications (and other browser-based applications). Here are some notes, links, and samples I have used to help me implement security correctly. 

<!--more-->

### Identity Providers

An important aspect of securing your Angular app is choosing an Identity Provider. IdentityServer4 is very popular. It allows you to host it yourself and to customize to your needs. It is an ASP.NET Core application, so it is necessary to support that type of application in your infrastructure. Auth0 and Okta are two hosted providers and are very popular. They may not be as customizable as IdentityServer, but they can provide a lot of value if you need to get a solution up and running quickly.

**IdentityServer4**  
[https://identityserver4.readthedocs.io/](https://identityserver4.readthedocs.io/)

**NOTE:** IdentityServer4 will no longer be a free product (as of Oct 1st 2020):  
[https://duendesoftware.com/products/identityserver](https://duendesoftware.com/products/identityserver)

**Auth0**  
[https://auth0.com/authenticate/angular/auth0/](https://auth0.com/authenticate/angular/auth0/)

**Okta**  
[https://developer.okta.com/code/angular/](https://developer.okta.com/code/angular/)

**OpenIddict**  
[https://github.com/openiddict](https://github.com/openiddict)  
[https://docs.orchardcore.net/en/dev/docs/reference/modules/OpenId/](https://docs.orchardcore.net/en/dev/docs/reference/modules/OpenId/)

### Courses/Tutorials

I have found these PluralSight courses extremely valuable. They each explained the topic well and included helpful samples and demos.

**Angular OAuth2 OIDC Configuration with IdentityServer4**  
[https://code-maze.com/angular-oauth2-oidc-configuration-identityserver4/](https://code-maze.com/angular-oauth2-oidc-configuration-identityserver4/)

**Securing Angular Apps with OpenID Connect and OAuth 2**  
by Brian Noyes  
[https://app.pluralsight.com/library/courses/openid-and-oauth2-securing-angular-apps/table-of-contents](https://app.pluralsight.com/library/courses/openid-and-oauth2-securing-angular-apps/table-of-contents)

**Securing ASP.NET Core 3 with OAuth2 and OpenID Connect**  
by Kevin Dockx  
[https://app.pluralsight.com/library/courses/securing-aspnet-core-3-oauth2-openid-connect/table-of-contents](https://app.pluralsight.com/library/courses/securing-aspnet-core-3-oauth2-openid-connect/table-of-contents)

Links

**OpenID Connect & OAuth 2.0 – Security Best Practices - Dominick Baier**  
[https://www.youtube.com/watch?v=jeRALmfyoqg](https://www.youtube.com/watch?v=jeRALmfyoqg)


