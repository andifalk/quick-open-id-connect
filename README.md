# Quick Open ID Connect

Fast Intro into federated identities with OpenID Connect hands-on.

What will we learn here:

* Get to know (install/configure) an IAM provider (in this case Keycloak)
* Implement a resource server using authentication with JWT (JSON Web Tokens)
* Login to Keycloak to get a JWT

## Setup

First, let's install and configure Keycloak.

1. Navigate to https://www.keycloak.org/downloads and download the Keycloak server Zip file
2. Extract Zip file
3. Open a terminal window and change directory to the one keycloak has been extracted into
4. Perform command `bin/kc.sh start-dev` on Linux/Mac or `bin\kc.bat start-dev` on Windows
5. Open http://localhost:8080 in your web browser
6. Create an administrative user, it is ok to just use admin/admin for simplicity (do not use this for production!!!)
7. Open Administration Console and login with the created user credentials, now you should see the admin console

## Configure Keycloak

1. First, create a new realm, select drop-down on the left and click `Create realm` with name `demo`.
2. Now create one user, go to `Users` menu and fill in the following user data and finally click `Create`:
   * Required user actions: No selection here
   * Username: `jdoe`
   * Email: `john.doe@example.com`
   * First name: `John`
   * Last name: `Doe`
   * Email verified: `On`
3. The user needs also a password, so switch to tab `Credentials` and set a password for this user, just use `secret` as password and switch `Temporary` to Off
4. Finally, we need to configure a client to login for getting a JWT to authenticate the resource server, select menu entry `Clients` for this and create a client:
   * Client type: `OpenID Connect`
   * Client ID: `demo-client`
   * `Next`
   * Client authentication: `Off`
   * Select `Standard flow` and `Direct access grants`
   * `Next`
   * Valid redirect URIs: `http://localhost:9095`
   * `Save`

With this configuration is finished.

## Implement the resource server

Open your Java IDE and create a new Java Spring Boot project with the following dependencies (you may  also use http://start.spring.io if your IDE does not have a spring boot wizard):

* Spring Web
* OAuth2 Resource Server

In the created project, first rename `application.properties` to `application.yml`, reformat the existing entry and add the following entry:

```yaml
server:
  port: 9090
```

As Keycloak runs on port 8080 (the default for spring boot projects), we need to reconfigure our project to port `9090`.

The next step is to configure the application as resource server with JWT authentication. Just add the following snippet to `application.yml`:

```yaml
spring: 
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: http://localhost:8080/realms/demo/protocol/openid-connect/certs
```

By adding the OAuth2 Resource Server dependency Spring security automatically configures the authentication via bearer token (JWT) and validates the JWT by:

* Validating the signature using the public key provided by keycloak
* Validating the expected JWT issuer
* Validating if the token is not expired

To validate the JWT signature our application needs to know where to load the public key from. This is done by the configuration of the `jwk-set-uri` above.
This information can be retrieved from the public OpenID Configuration endpoint http://localhost:8080/realms/demo/.well-known/openid-configuration of Keycloak (`jwks-uri` entry).

Finally, we need a sample REST API to test our JWT authentication. Create a new class `HelloApi` with the following contents:

```java
@RestController
public class HelloApi {

    @GetMapping("/hello")
    public String hello(@AuthenticationPrincipal Jwt jwt) {
        return "Hello " + jwt.getClaimAsString("given_name") + " " + jwt.getClaimAsString("family_name");
    }
}
```

Notice the annotation `@AuthenticationPrincipal` with the `JWT` class object. After successful validation of a specified JWT in the HTTP `Authorization` 
this object represents the JWT contents.

## Run & test the application

Now let's start the application and test it.

You may use a tool like `Postman` to get a JWT token from keycloak and call the API of the application at `http://localhost:9090/hello`.
In case you use IntelliJ as Java IDE you may use the provided HTTP request in folder `requests` of the reference solution in this project.

### Reference Documentation

For further reference, please consider the following sections:

* [OAuth2 Resource Server](https://docs.spring.io/spring-boot/docs/3.3.3/reference/htmlsingle/index.html#web.security.oauth2.server)