# Keycloak Quickstart
The quickest of quickstarts! 🚗💨

This repository should help you get a sample application secured using Keycloak
up and running in a couple of minutes.

Our example application included is designed to run on a separate domain to its
backend so we use _Bearer Only_ as our _Access Type_ when creating it in the
Keycloak Adminstration console. This is typically used by Single Page Applications,
Mobile Applications, and generally any applicatin that won't utilise Cookies.

## Prerequisites
* Node.js 6+
* npm 5+
* Docker (steps tested using 17.09.1-ce but other versions should be fine)


## Running Keycloak
Once you have a recent Docker version installed you can run Keycloak in the
background using the following command:

```
$ docker run -e DB_VENDOR=H2 -e KEYCLOAK_USER=admin -e \
KEYCLOAK_PASSWORD=password -p 8080:8080 --name keycloak-test -dit \
jboss/keycloak-openshift:3.4.3.Final
```

If you’d like to check logs just run:

```
$ docker logs keycloak-test
```

You can check the container status via:

```
$ docker ps | grep keycloak
```

Use the `docker stop keycloak-test` and `docker start keycloak-test` commands to
restart the server.

You can `docker stop keycloak-test` and `docker rm keycloak-test` to start from
a blank slate using `docker run` again.


## Configure Keycloak and a User Account
Check the `screenshots/` folder of this repository if you need a visual aid to
any steps here.

### Create a Realm
Navigate to http://localhost:8080 in a web browser (or whatever port you used 
with `docker run` above) and you should see the Keycloak home page:

Click the _Administration Console_ link and login using the `KEYCLOAK_USER` and `KEYCLOAK_PASSWORD` values passed to `docker run`.

Once logged in choose _Add Realm_ in the top left. Name it something simple
without spaces like "TestApp" then click _Create_. This represents our
application/project in Keycloak.

### Create a Client Application
Navigate to the _Clients_ section of your Realm and choose _Create_. Enter a
name and the _Root URL_ of http://localhost:3030 as shown below. The root URL is
where our application will be served from.

After your client is created change the _Access Type_ on the _Settings_ tab to
_Public_, then navigate to the `Installation` tab and choose the format
`Keycloak OIDC JSON`. Copy and paste the generated JSON into `www/keycloak.json`
file of this repository.

This client is a representation of our web application in Keycloak.

### Create a Backend Application
Similar to the last step we need to create a backend application representation.
Create a new _Client_ with the same information (http://localhost:3030), but
this time instead of choosing _Public_ for _Access Type_ choose _Bearer Only_.
Navigate to the _Installation_ tab and download the JSON again, but this time
paste it into the `index.js` file to replace the existing `keycloakConf`.

This client is a representation of our backend API in Keycloak and is required
to secure our API endpoints using the `keycloak-connect` Node.js module's
`keycloak.protect()` function.

### Create a User Account
Finally, we need to create a user account to login. Navigate to the _Users_
section and choose _Add User_ in the top right. Enter some details, add 
_Update Password_ to the _Required User Actions_ section, and choose save. Once
the user is saved, view the user and open their _Credentials_ tab and set a new
password for the user account. This password is required for initial logins.


## Using Keycloak with our Application

Now that a _Realm_ and user are configured we can actually use Keycloak with our
application. Navigate to `http://localhost:8080/auth/js/keycloak.js` and
download the script that is loaded into `www/keycloak.js` in this repository.

We're ready to try out Keycloak now, so type `npm install` then `npm start` in
the root of this repository and open `http://localhost:3030` once prompted.

Click the login button, then enter the password you set for the user. Upon login
you will be prompted to set a new password. Once the new password is set you'll
be redirected to the application at `http://localhost:3030` and this time it
will display session information.

The XHR test in the web page hits the `http://localhost:3030/ping-protected`
endpoint. You can try hit this yourself using curl or a web browser, but you'll
be blocked unless you include a valid header like so:

```
$ curl -H 'authorization: Bearer $(KEYCLOAK_JWT)' http://localhost:3030/ping-protected
```

The value of `KEYCLOAK_JWT` can be gotten by logging in on the web page in this
sample and copying the token loaded.

Fin.
