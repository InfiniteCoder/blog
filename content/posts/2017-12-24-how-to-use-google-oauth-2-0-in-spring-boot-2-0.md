---
date: "2017-12-24T00:00:00Z"
gh-badge: fork
gh-repo: InfiniteCoder/SpringBoot2OAuth2
published: true
subtitle: Using Spring Boot 2.0 and Spring Framework 5.0 features to simplify OAuth
  2.0 Login
title: How to use Google OAuth 2.0 in Spring Boot 2.0
---

> **While I have used Google here, the process is pretty much same for Facebook, Github and Okta.**

Spring Boot 2.0 is coming soon, and the first Release Candidate is already out. With it, it brings about a lot of changes. Spring Boot now [supports and requires Spring Framework 5.0](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0.0-M1-Release-Notes#spring-framework-50), which also has changed a lot.

Now, it is much easier to set up Google as OAuth 2.0 client, with much less configuration required. This post shall explain how you can set up OAuth 2.0 client in Spring Boot 2.

## Getting the credentials
Head on to [google developer console](https://console.developers.google.com/), register new application( if you haven't already) and get the **client id** and **client secret**. On the consent page, make sure you set the redirection url to `http://localhost:8080/login/oauth2/code/google`. That is, the url is `{baseUrl}/login/oauth2/code/{registrationId}`

> **Note that the default redirection URL is not `{baseUrl}/login`, as was the case with previous versions.**


## Setting up the dependencies
Now create a new project, preferably using [spring initializr](https://start.spring.io/). Make sure you select spring boot 2.0. There are six dependencies you will need, beyond the obvious ones(like web-starter).
For spring security, you need to add **`spring-security-config`**, **`spring-security-core`**, and **`spring-security-web`**, all of which are under **`org.springframework.security`** group.
Also, for spring security oauth2, you need to add **`org.springframework.security`**, **`spring-security-oauth2-client`**, and **`spring-security-oauth2-jose`**, also user the same group.

Your `pom.xml` dependency section should look something like this,

``` xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- spring security -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>5.0.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
            <version>RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>5.0.0.RELEASE</version>
        </dependency>

        <!-- Dependencies for OAuth 2.0-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-core</artifactId>
            <version>5.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-client</artifactId>
            <version>5.0.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-jose</artifactId>
            <version>5.0.0.RELEASE</version>
        </dependency>
</dependencies>
```

## Setting up the configuration
In your `application.yml` file, add this configuration

``` yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${client-id}
            client-secret: ${client-secret}
```

Either replace the `${client-id}` and `${client-secret}` with actual values (not recommended) or add those two as environment variables.

> **Note that we have set up registrationId as google. This lets spring security autoconfigure other details, which don't change often.**


## That's it
We are done. Boot up the application, and launch `http://localhost:8080/`. You should get redirected to the login page, which should show a link that reads google. Click it, you should be redirected to google login page, and upon login, back to `index.html`.

For a complete app, check out my [repository on github](https://github.com/InfiniteCoder/SpringBoot2OAuth2).

> **Read [spring security reference](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#jc-oauth2login) for more details.**
