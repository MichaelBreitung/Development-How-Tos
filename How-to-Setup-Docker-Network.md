# How to Setup a Simple Docker Network

Consider a very simple use case: You have two docker containers running on the same host machine. One container is running a client app which must connect to a server running in the other container. As an example, you could have a Node.js app running in one container and a Redis server or some database running in the other container.

The problem is that out of the box, the app in the first container will not be able to connect to the server in the second container.

There are different approaches to make this work.

## host.docker.internal

This approach is possible without additional packages like *Docker Compose*.

1) Start the container running the server and use port-forwarding to allow incoming connections to the contained server:

   ````
   docker container run -p < external port >:< server listen port > < server container name | ID >
   ````

2) Configure your client app to use the following address:

   ````
   host: host.docker.internal
   port: < external port >
   ````

   Note: The *external port* is the one provided via port forwarding when starting the container that runs the server.

3) Start the container running the client with *host.docker.internal*:

   ````
   docker container run --add-host host.docker.internal:host-gateway < client container name | ID >
   ````

## Docker Compose

You can use the Docker Compose plugin to achieve the same in a simpler and more flexible manner. Define a *docker-compose.yml* file in which you specify the containers that should run in the same Network. You will get the above behavior out of the box without the need for *host.docker.internal*.

Here's a simple *docker-compose.yml*:

````
version: '3'

services:
  server-app:
    image: < server container name >
    hostname: server
  client-app:
    image: < client container name >
    hostname: client
````

Specify a *hostname* so you can use it when creating connections between your containers. There's no need to use IPs. This makes it easy to setup complex multi-container applications.

## References

- https://medium.com/@TimvanBaarsen/how-to-connect-to-the-docker-host-from-inside-a-docker-container-112b4c71bc66

