# Introduction

This project exposes a simple REST endpoint where the service `greeting` is available, but properly secured, at this
address `http://hostname:port/greeting` and returns a json Greeting message after the application issuing the call to
the REST endpoint has been granted to access the service.

```
{
    "content": "Hello, World!",
    "id": 1
}

```

The id of the message is incremented for each request. To customize the message, you can pass as parameter the name of
the person that you want to send your greeting.

To manage the security, roles & permissions to access the service, a 
[Red Hat SSO](https://access.redhat.com/documentation/en/red-hat-single-sign-on/7.0/securing-applications-and-services-guide/securing-applications-and-services-guide)
backend will be installed and configured for this project.

It relies on the Keycloak project which implements the `OpenId` connect specification which is an extension of the
`Oauth2` protocol.

After a successful login, the application will receive an `identity token` and an `access token`. The identity token
contains information about the user such as username, email, and other profile information.

The access token is digitally signed by the realm and contains access information (like user role mappings)

This is typically this `access token` formatted as a JSON Token that the Vert.x application will use to allow access to
the application.

The configuration of the adapter is defined within the `app/src/main/java/org/obsidiantoaster/quickstart/RestApplication.java`
file using these environment properties:

```
$REALM
$REALM_PUBLIC_KEY
$SSO_HOST
$CLIENT_APP
$CLIENT_SECRET
```

Note that the config object is what you would download from the Keycloak admin console. Just for simplicity we encode it
in the source code but you could just load the json at runtime.

One can request a token manually using the `OpenID-Connect` protocol. A typical request would be: 

```
https://<SSO_HOST>/auth/realms/<REALM>/protocol/openid-connect/token?client_secret=<SECRET>&grant_type=password&client_id=CLIENT_APP
```

And the HTTP requests accessing the endpoint/Service will include the Bearer Token

```
http://<Vert.x_App>/greeting -H "Authorization:Bearer <ACCESS_TOKEN>"
```

Alternatively one can use helper libraries has for example in the html demo `app/src/main/resources/webroot/index.html`
where the official `keycloak.js` is used to simplify the interaction with the server.

The project is split into two Apache Maven modules - `app` and `sso`.

The goal of this project is to deploy the quickstart against an OpenShift environment (online, dedicated, ...).

# Prerequisites

To get started with these quickstarts you'll need the following prerequisites:

Name | Description | Version
--- | --- | ---
[java][1] | Java JDK | 8
[maven][2] | Apache Maven | 3.2.x
[oc][3] | OpenShift Client | v3.3.x
[git][4] | Git version management | 2.x

[1]: http://www.oracle.com/technetwork/java/javase/downloads/
[2]: https://maven.apache.org/download.cgi?Preferred=ftp://mirror.reverse.net/pub/apache/
[3]: https://docs.openshift.com/enterprise/3.2/cli_reference/get_started_cli.html
[4]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

In order to build and deploy this project, you must have an account on an OpenShift Online (OSO): https://console.dev-preview-int.openshift.com/ instance.

# OpenShift Online

1. Using OpenShift Online or Dedicated, log on to the OpenShift Server.

    ```bash
    oc login https://<OPENSHIFT_ADDRESS> --token=MYTOKEN
    ```

1. Create a new project on OpenShift.

    ```bash
    oc new-project <some_project_name>
    ```

1. Build the quickstart.

    ```
    mvn clean install -Popenshift
    ```

# Deploy the Application

1. First, to deploy Red Hat SSO, clone the [redhat-sso](https://github.com/obsidian-toaster-quickstarts/redhat-sso) project
and following the README.md instructions.

    ```bash
    cd redhat-sso
    mvn fabric8:deploy
    ```

1. Open the OpenShift web console to see the status of the app and the exact routes used to access the app's greeting endpoint, or to access the Red Hat SSO's admin console.

    Note: until [CLOUD-1166](https://issues.jboss.org/browse/CLOUD-1166) is fixed,
    we need to fix the redirect-uri in RH-SSO admin console, to point to our app's route.

1. To specify the Red Hat SSO URL to be used by the Spring Boot application,
you must change the SSO_URL env variable assigned to the DeploymentConfig object.

    Note: You can retrieve the address of the SSO Server by issuing this command `oc get route/secure-sso` in a terminal and get the HOST/PORT name

    ```
    oc env dc/secured-vertx-rest SSO_URL=https://secure-sso-sso.e8ca.engint.openshiftapps.com/auth
    ```

# Access the service

Use the rh-sso project scripts to access the secured endpoints.

TODO:
1. cd redhat-sso
1. scripts/token_req.sh secured-swarm-rest

# Access the service using a user without admin role

TODO.

The patterns property defines as pattern, the `/greeting` endpoint which means that this endpoint is protected by
Keycloak. Every other endpoint that is not explicitly listed is NOT secured by Keycloak and is publicly available.

To verify that a user without the `admin` role can't access the service, you will create a new user using the following
bash script

```
./scripts/add_user.sh <SSO_HOST> <Vert.x_HOST>
```

Next, you can call again the greeting endpoint by issuing a HTTP request where the username is `bburke` and the password
`password`. In response, you will be notified that yoou can't access to the service

```
./scripts/token_user_req.sh <SSO_HOST> <Vert.x_HOST>
```
