---
layout: default
title: Download
rightmenu: false
permalink: /download
---

{% include notificationBanner.html %}

<h1 class="no-margin-top">A lightweight REST API library</h1>

Javalin is a true microframework with only one required dependency: SLF4J (logging).

By default Javalin also depends on Jetty, but you can exclude it (or use the `javalin-without-jetty` artifact)
if you want to use a different Webserver (like Tomcat, etc).

Javalin also has plugins for JSON mapping, template rendering, and OpenAPI (Swagger), but they're
optional dependencies that you have to add manually.

## Download Javalin
{% include macros/mavenDep.md %}

## Javalin modules

There are multiple modules available for Javalin.
All modules are released in sync with the core `javalin` module, you just have to replace
the artifact id:

* `javalin` - the core Javalin dependency (shown in the snippet above)
* `javalin-bundle` - `javalin`, plus `javalin-openapi`, `jackson` and `logback` (good to get started fast)
* `javalin-openapi` - the OpenAPI plugin and all required dependencies
* `javalin-graphql` - the new GraphQL plugin
* `javalin-without-jetty` - `javalin` with all `jetty` dependencies excluded
  and Jetty specific methods removed (useful for running on e.g. Tomcat)

### Manual downloads
You can get the prebuilt jar from [Maven Central](https://repo1.maven.org/maven2/io/javalin/javalin/).\\
You can get the source on [GitHub](https://github.com/javalin/javalin), or [download it as a zip](https://github.com/javalin/javalin/archive/master.zip).
