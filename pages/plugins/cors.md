---
layout: docs
title: CORS plugin documentation
rightmenu: false
permalink: /plugins/cors
---

<div id="spy-nav" class="right-menu" markdown="1">
- [Getting Started](#getting-started)
- [Allowing any host](#allowing-any-host)
- [Subdomain wildcard support](#subdomain-wildcard-support)
- [Multiple CORS configurations](#multiple-cors-configurations)
- [allowCredentials and exposeHeader](#allowcredentials-and-exposeheader)
</div>

<h1 class="no-margin-top">CORS plugin</h1>

The CORS plugin manages CORS related headers for you given some configuration such as allowed hosts.

## Getting started

You can enable the cors plugin through the `config.plugins` part of the config:

```java
Javalin.create(config -> {
    config.plugins.enableCors(cors -> {
        cors.add(it -> {
            it.allowHost("example.com", "javalin.io");
        });
    });
});
```

This example would allow the origins `https://example.com` and `https://javalin.io`.

The default scheme can be changed by setting it: `defaultScheme = "http"`, but you can also just specify it with your 
`allowHost()` call: `allowHost("http://example.com")`

## Allowing any host

Allowing everybody by using can be done with `anyHost()`.
`anyHost()` adds the special star origin `*`, allowing any host from a CORS standpoint. Do note that you can still
deny via other means such as an access manager, but it is recommended to just allow the hosts that you need.

```java
Javalin.create(config -> {
    config.plugins.enableCors(cors -> {
        cors.add(it -> {
            it.anyHost();
        });
    });
});
```

Similar to `anyHost()` you can set `reflectClientOrigin = true` to reflect back the clients origin instead of the generic
star. This has the same implications as `anyHost()`, so it should be considered carefully.

```java
Javalin.create(config -> {
    config.plugins.enableCors(cors -> {
        cors.add(it -> {
            it.reflectClientOrigin = true;
        });
    });
});
```

## Subdomain wildcard support

Special support for subdomains is added by allowing a single star as a wildcard.
`allowHost("*.example.com")` would allow any subdomain of `example.com` to access your resources from a CORS standpoint.

```java
Javalin.create(config -> {
    config.plugins.enableCors(cors -> {
        cors.add(it -> {
            it.allowHost("*.example.com");
        });
    });
});
```

## Multiple CORS configurations

It is also possible to have different cors configurations for different paths. 
Take a look at the following example:

```java
Javalin.create(config -> {
    config.plugins.enableCors(cors -> {
        cors.add(it -> {
            it.path = "images*"
            it.allowHost("https://images.local");
        });
        cors.add(it -> {
            it.path = "videos*"
            it.allowHost("https://videos.local");
        });
        cors.add(it -> {
            it.path = "music*"
            it.allowHost("https://music.local");
        });
    });
});
```

Everything listed under `images` would be only accessible by the host `images.local` and everything under `videos` 
only to the host `videos.local`.

## allowCredentials and exposeHeader

For those who need it you can also set the `ACCESS_CONTROL_ALLOW_CREDENTIALS` header by setting `allowCredentials = true`
and expose headers to the website's JavaScript by using e.g. `exposeHeader("x-server")` to expose the `x-server` header.

```java
Javalin.create(config -> {
    config.plugins.enableCors(cors -> {
        cors.add(it -> {
            it.allowHost("*.example.com");
            it.allowCredentials = true;
            it.exposeHeader("x-server");
        });
    });
});
```