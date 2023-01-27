# How To Use Docker

## Helpful Commands

### docker image

- **docker image pull < name:tag >** - Pull an image with a certain tag from the docker registry.
- **docker image build -t < name:tag > .** - Call this command inside a folder containing a *Dockerfile* to create an image. The final . tells the command to look in the current folder for the *Dockerfile*. You can also specify a specific path there and call this command from another directory.
- **docker image ls** - list all installed images. Shorthand is **docker images**.
- **docker image rm < name | ID >** - Remove specific image. Shorthand is **docker rmi**.
- **docker image rm $(docker images -q)** - Remove all images.

### docker container

- **docker container run -d < name | ID >** - Create and start a container of specified name or ID and detach from it. You stay inside your current session and keep control over the input.
- **docker container run -it < name | ID > bash** - Create and start a container of specified name or ID and run bash inside of it in interactive mode. This allows you to directly interact with the bash. Shorthand is **docker run -i < name | ID > bash**
- **docker container run --network < network name > < name | ID >** - Create and start a container and attach it to a specific network.
- **docker container exec -it < name | ID > bash** - Run bash inside an already started container of specified name or ID in interactive mode. 
- **docker container ls -a** - list all running containers and show current status. Shorthand is **docker ps -a**. With this command you also see containers that have been stopped and their names.
- **docker container stop < name | ID >** - Stops a running container. Shorthand ist **docker stop**. Note that the containers file system still exists.
- **docker container stop $(docker ps -a -q)** - Stops all running containers.
- **docker container rm < name | ID >** - Removes a containers data. You can use *docker ps -a* to see, which containers are already stopped. Those are the ones you can also remove, to free some disk space. Mind that you'll loose all data as the file system is deleted.
- **docker container rm -f $(docker ps -a -q)** - Removes data of all containers. Make sure to first stop the containers.
- **docker container inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' < name | ID>** - List networks a container belongs to by their network name.
- **docker container inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' < name | ID >** - Get the IP Address of a running container.

### docker network

- **docker network inspect -f '{{range .Containers}}{{.Name}} {{end}}' < network name >** - List all containers belonging to Network with name *name*.

### docker system

- **docker system prune** - remove all containers and cleanup dangling images

**Note**: If you specify a container by ID, it's sufficient to use the first few characters and numbers of that ID. 

**Note**: Docker will give containers automatically generated names. If you want to specify a custom name, use the *--name* option when calling *docker container run*.

## Use Cases

### Node App

Consider you want to run a Node applicatoin inside a docker container. To remain flexible during the development of the application while using docker to run the app, you will likely want to use *Volumes* to mount your local app code into the docker container.

The *node_modules* that are required to run the app in the container, should only be installed in the container though and not in your local file system.

For this to work under Linux, you must ensure that the permissions inside the container are setup properly as explained in [this article](https://www.digitalocean.com/community/tutorials/how-to-build-a-node-js-application-with-docker) under step 3.

A Dockerfile might look like this:

    FROM node:16-alpine
    
    # act as node user, which should be available on node alpine containers
    USER node
   
    # create the app folder inside the node users space
    RUN mkdir -p /home/node/app
    
    # set the workdir to the newly created folder
    WORKDIR /home/node/app
     
    # get the package.json file and install node modules inside workdir
    # user and group for those files will be node as per chown option
    COPY --chown=node:node ./package.json ./
    RUN npm install
    
    # if you use Volumes, you can skip the following line
    COPY --chown=node:node ./ ./
     
    CMD ["npm", "start"]
    
To run the container using volumes, use the following command:

    docker container run -p < port forwarding for app > -v /home/node/app/node_modules -v $(pwd):/home/node/app < name | ID >
    
As an alternative, you can use *Docker Compose*:

    version: '3'

    services:
      node-app:
        build: .
        ports: 
          - < port forwarding for app >
        volumes:
          - /home/node/app/node_modules
          - .:/home/node/app
