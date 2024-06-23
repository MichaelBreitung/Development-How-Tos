# How to Setup Keycloak

## Quick Setup

1. Create a new Realm

   - provide a _name_, which will be used on client side to connect to Keycloak

2. Creata a Client inside the Realm

   - provide _clientId_ which will be used on client side to connect to Keycloak

   - Setup Redirect Urls and Web origins - for example, for your Web application

3. Create a User and provide initial credentials

4. Use *keycload-js* inside your JavaScript client to setup authentication:
   ````
   const keycloak = new Keycloak({
     url,
     realm,
     clientId,
   });
   ````

   In the _url_, include the port on which Keycloak is listening. This is per default 8080.

5. A call to _keycloak.init_ will finish the setup in the client and redirect a user to Keycloak, if he/she is not already logged in.

## Client

- Redirect Urls - http://localhost/*
  It is important to include the /* at the end. Just using http://localhost, for example, will give a configuration error.
- Web origins - http://localhost
  This is the CORS setup, which should be http://localhost, if your app runs on Port 80, or include the port. For example, http://localhost:3000 for the detault React SPA.
- fdsf

### CORS

- https://github.com/jangaraj/keycloak-cors-issue-debugging

## User

Under the users tab, you can create users and specify the first actions the users should do when they are first redirected to Keycloak by using the client.

If you click on an individual user, you can also sign him/her out under the sessions tab. An overview over the sessions of all users can also be found under the respective client, under the _Sessions_ tab.



## References

- React and Keycloak - https://youtu.be/5z6gy4WGnUs
- Configuration Guide - https://medium.com/@bcarunmail/securing-rest-api-using-keycloak-and-spring-oauth2-6ddf3a1efcc2s

