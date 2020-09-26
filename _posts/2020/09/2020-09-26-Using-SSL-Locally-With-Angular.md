---
layout: post
title: 'Using SSL Locally With Angular'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In this day and age, it is required that all websites use SSL. So I want to ensure I am using it wherever possible. When developing locally with Angular, I wanted to use https://localhost:4200/. I found a couple of ways to help, but it didn't go exactly as planned.

<!--more-->

I found several articles that claimed to do the voodoo I wanted. all of them revolved around creating a self-signed SSL certificate I could use for local development:

**Develop Locally with HTTPS, Self-Signed Certificates and ASP.NET Core**  
[https://www.humankode.com/asp-net-core/develop-locally-with-https-self-signed-certificates-and-asp-net-core](https://www.humankode.com/asp-net-core/develop-locally-with-https-self-signed-certificates-and-asp-net-core)

**Developing locally with ASP.NET Core under HTTPS, SSL, and Self-Signed Certs**  
[https://www.hanselman.com/blog/DevelopingLocallyWithASPNETCoreUnderHTTPSSSLAndSelfSignedCerts.aspx](https://www.hanselman.com/blog/DevelopingLocallyWithASPNETCoreUnderHTTPSSSLAndSelfSignedCerts.aspx)

**Debug Angular apps locally with a self signed SSL certificate**  
[https://medium.com/medialesson/debug-angular-apps-locally-with-a-self-signed-ssl-certificate-98191581bc83](https://medium.com/medialesson/debug-angular-apps-locally-with-a-self-signed-ssl-certificate-98191581bc83)

Unfortunately, none of them worked. Google Chrome and Microsoft Edge both rejected the certificate and wouldn't let me use it locally with https://localhost:4200. I reasoned that I was doing something wrong, or the advice was out of date and the latest versions of Windows, and the web browsers was somehow the cause.

I found a new approach here:

**Serve Angular App Over HTTPS (Localhost – Angular CLI)**  
[https://fmoralesdev.com/2020/01/03/serve-angular-app-over-https-using-angular-cli/](https://fmoralesdev.com/2020/01/03/serve-angular-app-over-https-using-angular-cli/)

And it didn't work perfectly either. But I did get it to work however. I ended up running this command from a WSL session:

```shell
openssl req -newkey rsa:2048 -x509 -nodes -keyout server.key -new -out server.crt -config ./openssl-custom.cnf -sha256 -days 365
```

The resulting `server.key` and `server.crt` files worked perfectly with `ng serve`. I copied the files into an `ssl` folder in my angular application and ran the following command:

```shell
ng serve --ssl true -o --sslKey ssl/server.key --sslCert ssl/server.crt
```

And as it turns out, this can also be used to serve up my blog locally using Jekyll:

**Running SSL on localhost**  
[https://remotesynthesis.com/blog/running-ssl-localhost](https://remotesynthesis.com/blog/running-ssl-localhost)

```shell
jekyll serve --drafts --ssl-key ssl/server.key --ssl-cert ssl/server.crt
```

The Certificate is only good for one year. So I'll probably have do to this all over again this time next year. Hopefully some of this information will still be applicable!

