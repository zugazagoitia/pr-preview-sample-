---
layout: tutorial
official: true
title: Testing Javalin Applications
permalink: /tutorials/testing
summarytitle: Testing Javalin Applications
summary: Learn how to run different kinds of tests in Javalin. Unit tests, functional/integration tests, UI/end-to-end tests.
date: 2020-01-18
author: <a href="https://www.linkedin.com/in/davidaase" target="_blank">David Åse</a>
language: ["java", "kotlin"]
github: https://github.com/javalin/javalin-samples/tree/main/javalin5/javalin-testing-example
---

## Introduction
Since Javalin is a library, there are no requirements for how tests must be written.
This guide will outline a few common approaches. None of the approaches are better
than the others, you just have to find something that works for you.

To begin, you'll need to have a Maven project configured [(→ Tutorial)](/tutorials/maven-setup)
with Javalin and [AssertJ](https://joel-costigliola.github.io/assertj/):

```markup
<dependency>
    <groupId>io.javalin</groupId>
    <artifactId>javalin-bundle</artifactId>
    <version>{{site.javalinversion}}</version>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.11.1</version>
    <scope>test</scope>
</dependency>
```

## Unit tests
Unit tests are tests for the smallest and most isolated part of an application.
In Javalin, this means testing anything that implements the `Handler` interface.
Unit tests are very fast and cheap to run, and they usually require
[mocking](https://en.wikipedia.org/wiki/Mock_object) of objects.
To begin, we will need to add a Mocking library.

For Java the most popular choice is [Mockito](https://site.mockito.org/):
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.6.1</version>
    <scope>test</scope>
</dependency>
```

For Kotlin, the most poplar choice is [MockK](https://mockk.io/):
```xml
<dependency>
    <groupId>io.mockk</groupId>
    <artifactId>mockk</artifactId>
    <version>1.12.5</version>
    <scope>test</scope>
</dependency>
```

Once we have the mocking library added, we'll mock the Javalin `Context`, since
the `Context` class is responsible for input and output in Javalin `Handler`s.
We're using a static/singleton controller in this example for simplicity, but
how you structure that code is entirely up to yourself.

{% capture java %}
import io.javalin.http.BadRequestResponse;
import io.javalin.http.Context;
import org.junit.Test;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;
import static org.mockito.Mockito.verify;

public class UnitTest {

    private final Context ctx = mock(Context.class);

    @Test
    public void POST_to_create_users_gives_201_for_valid_username() {
        when(ctx.queryParam("username")).thenReturn("Roland");
        UserController.create(ctx); // the handler we're testing
        verify(ctx).status(201);
    }

    @Test(expected = BadRequestResponse.class)
    public void POST_to_create_users_throws_for_invalid_username() {
        when(ctx.queryParam("username")).thenReturn(null);
        UserController.create(ctx); // the handler we're testing
    }

}
{% endcapture %}
{% capture kotlin %}
import io.javalin.http.BadRequestResponse
import io.javalin.http.Context
import io.mockk.every
import io.mockk.mockk
import io.mockk.verify
import org.junit.Test

class UnitTest {

    private val ctx = mockk<Context>(relaxed = true)

    @Test
    fun `POST to create users gives 201 for valid username`() {
        every { ctx.queryParam("username") } returns "Roland"
        UserController.create(ctx) // the handler we're testing
        verify { ctx.status(201) }
    }

    @Test(expected = BadRequestResponse::class)
    fun `POST to create users throws for invalid username`() {
        every { ctx.queryParam("username") } returns null
        UserController.create(ctx) // the handler we're testing
    }

}

{% endcapture %}
{% include macros/docsSnippetKotlinFirst.html java=java kotlin=kotlin %}

In the first test, we instruct the `Context` mock to return `"Roland"` when
`queryParam("username")` is called. After that, we call the `Handler` (`UserController.create(ctx)`), and then we verify
that `ctx.status(201)` was called. We could also have mocked the `UserController`
to verify that the user was added.

In the second test, we return `null` for the `username`, and we make sure
to *expect* a `BadRequestResponse`.

The code for `UserController.create(ctx)` looks like this:

{% capture java %}
public static void create(Context ctx) {
    String username = ctx.queryParam("username");
    if (username == null || username.length() < 5) {
        throw new BadRequestResponse();
    } else {
        users.add(username);
        ctx.status(201);
    }
}
{% endcapture %}
{% capture kotlin %}
fun create(ctx: Context) {
    val username = ctx.queryParam("username")
    if (username == null || username.length < 5) {
        throw BadRequestResponse()
    } else {
        users.add(username)
        ctx.status(201)
    }
}
{% endcapture %}
{% include macros/docsSnippetKotlinFirst.html java=java kotlin=kotlin %}

The two tests cover the branches of the if/else. There isn't a lot more to say about mocking that's specific to Javalin.
You can follow any general mocking tutorial for your favorite language and library.

## Functional/integration tests
Functional tests are "black box" tests, and only focus on the business requirements of an application.
In the unit tests (in the previous section), we mocked the `Context` object and called `verify`
to ensure that `ctx.status(201)` was called inside the `UserController.create(ctx)` `Handler`.
In functional tests, we just verify that we get the expected output for the provided input.
The easiest way of writing this type of test in Javalin is to use
the `javalin-testtools`, which comes included in the `javalin-bundle`:

{% capture java %}
import io.javalin.Javalin;
import io.javalin.plugin.json.JavalinJackson;
import io.javalin.testtools.JavalinTest;
import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class FunctionalTest {

    Javalin app = new JavalinTestingExampleApp("someDependency").javalinApp(); // inject any dependencies you might have
    private final String usersJson = new JavalinJackson().toJsonString(UserController.users);

    @Test
    public void GET_to_fetch_users_returns_list_of_users() {
        JavalinTest.test(app, (server, client) -> {
            assertThat(client.get("/users").code()).isEqualTo(200);
            assertThat(client.get("/users").body().string()).isEqualTo(usersJson);
        });
    }

}
{% endcapture %}
{% capture kotlin %}
import io.javalin.plugin.json.JavalinJackson
import io.javalin.testtools.JavalinTest
import org.assertj.core.api.Assertions.assertThat
import org.junit.Test

class FunctionalTest {

    private val app = JavalinTestingExampleApp("someDependency").app // inject any dependencies you might have
    private val usersJson = JavalinJackson().toJsonString(UserController.users)

    @Test
    fun `GET to fetch users returns list of users`() = JavalinTest.test(app) { server, client ->
        assertThat(client.get("/users").code).isEqualTo(200)
        assertThat(client.get("/users").body?.string()).isEqualTo(usersJson)
    }

}
{% endcapture %}
{% include macros/docsSnippetKotlinFirst.html java=java kotlin=kotlin %}

In Javalin's test suite, almost all of the tests are written like this.
I personally prefer this approach for tests, as each test touches the whole system, and you don't
risk making mistakes while manually specifying expected behavior (mocking). Javalin's test
suite starts and stops more than 500 Javalin instances, and running all the tests takes about ten seconds total
(most of those ten seconds is spent waiting for a WebSocket test and starting Chrome for browser tests).

## End-to-end/UI/scenario tests
Like functional tests, end-to-end tests focus on input and output, but typically describe a
longer scenario. For example, a user visiting a website, clicking on a link, filling in a form,
and submitting. These types of tests are usually written with Selenium in Java/Kotlin. Selenium
requires you to have the browser that you want to use installed, but it's also possible to
install that browser on demand. We'll need to add two dependencies:
[Selenium](https://selenium.dev/documentation/en/) and [WebDriverManager](https://github.com/bonigarcia/webdrivermanager):

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-chrome-driver</artifactId>
    <version>4.3.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.2.3</version>
    <scope>test</scope>
</dependency>
```

{% capture java %}
import io.github.bonigarcia.wdm.WebDriverManager;
import io.javalin.Javalin;
import io.javalin.testtools.JavalinTest;
import org.junit.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

import static org.assertj.core.api.Assertions.assertThat;

public class EndToEndTest {

    Javalin app = new JavalinTestingExampleApp("someDependency").javalinApp(); // inject any dependencies you might have

    @Test
    public void UI_contains_correct_heading() {
        JavalinTest.test(app, (server, client) -> {
            WebDriverManager.chromedriver().setup();
            ChromeOptions options = new ChromeOptions();
            options.addArguments("--headless");
            options.addArguments("--disable-gpu");
            WebDriver driver = new ChromeDriver(options);
            driver.get(client.getOrigin() + "/ui");
            assertThat(driver.getPageSource()).contains("<h1>User UI</h1>");
            driver.quit();
        });
    }

}
{% endcapture %}
{% capture kotlin %}
import io.github.bonigarcia.wdm.WebDriverManager
import io.javalin.testtools.JavalinTest
import org.assertj.core.api.Assertions.assertThat
import org.junit.Test
import org.openqa.selenium.WebDriver
import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.chrome.ChromeOptions

class EndToEndTest {

    private val app = JavalinTestingExampleApp("someDependency").app // inject any dependencies you might have

    @Test
    fun `UI contains correct heading`() = JavalinTest.test(app) { server, client ->
        WebDriverManager.chromedriver().setup()
        val driver: WebDriver = ChromeDriver(ChromeOptions().apply {
            addArguments("--headless")
            addArguments("--disable-gpu")
        })
        driver.get("${client.origin}/ui")
        assertThat(driver.pageSource).contains("<h1>User UI</h1>")
        driver.quit()

    }

}
{% endcapture %}
{% include macros/docsSnippetKotlinFirst.html java=java kotlin=kotlin %}

In this example, we just do `assertThat(driver.pageSource).contains("<h1>User UI</h1>")`, since writing
proper Selenium tests is outside of the scope of this guide.
You can use Selenium to simulate any type of user behavior. Have a look at the
[Selenium docs](https://selenium.dev/documentation/en/) for details.
The [WebDriverManager docs](https://github.com/bonigarcia/webdrivermanager) includes an example of
how to re-use your driver between multiple tests.


## Conclusion
Hopefully this brief guide has given you some ideas on how to test your Javalin application.
Since Javalin is just a library, you're more or less free to test however you like;
there is no "Javalin way" of testing.
