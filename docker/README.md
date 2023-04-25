#### Prerequisites

Install Docker 

- Mac via Homebrew

```dockerfile
$ brew install docker
```

- Windows via Chocolatey

```dockerfile
$ choco install docker 
```



#### Creating a Docker Image

What if we want to containerize something we created and publish it on Container Registry,  Quay.io or Dockerhub? 

What would that require? 

A `Dockerfile`.

A `Dockerfile` is a ["text document that contains all the commands you would normally execute manually in order to build a Docker image."](https://docs.docker.com/glossary/). 

- Create in a new folder in `~/workspace/` named `dockerfile-example`, cd into `dockerfile-example`. Next, `touch Dockerfile && code .`

- We will populate the `Dockerfile` with:

```dockerfile
FROM alpine

CMD ["echo", "Hello, world! This is a simple Dockerfile using echo."]
```

- In order to build the image we begin with:

```shell
$ docker build --no-cache -t simple-dockerfile-example .
$ docker run simple-dockerfile-example
A simple Dockerfile
```

While this is helpful for getting the basic workflow of `docker build` / `docker run` it is more instructive to see an application be built. For this example, we will adapt from the [Node with Docker Official Guide](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/).

- To demonstrate how fast it is to go from project to containerized application we begin with a [Express app](https://expressjs.com/en/starter/generator.html) :

```shell
$ npx express-generator app-to-containerize
$ cd app-to-containerize && npm install
```

- inside of `views/index.jade` we add a single line:

```
extends layout

block content
  h1= title
  p Welcome to #{title}
  p This is a Node Application that has been containerized with Docker!
```

- next, we create a Dockerfile with `touch Dockerfile`. First, let's use Google or Dockerhub to see if there are any existing images we can use as a _base image_ for our build.

- The [Official NodeJS](https://hub.docker.com/_/node) image should work:

```dockerfile
## Use node Docker image, version 16-alpine
FROM node:16-alpine

## From the documentation, "The WORKDIR instruction sets the working directory (folder) for any
## RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile"
WORKDIR /usr/src/app

## COPY package.json and package-lock.json into root of WORKDIR
COPY package*.json ./

## Executes commands
RUN npm install

## Copies files from source to destination, in this case the root of the build context
## into the root of the WORKDIR
COPY . .

## Documents that the container process listens on port 3000
EXPOSE 3000

## Command to use for starting the application
CMD ["npm", "start"]
```

One important concept in the above example is the [build context](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#understand-build-context). 

In the simplest terms possible, the build context is the folder where we run `docker` commands. 

Our `Dockerfile` will typically be located at the same level of the file tree as the build context.

- Since our build context, for this example, includes a folder of installed dependencies, `node_modules/`, we can use `.dockerignore` to ensure they are not copied into the image:

```docker
node_modules/
```

- Next we build the image and start a container with this image:

```shell
$ docker build --no-cache -t express-example-app .
$ docker run --rm  -p 8080:3000 express-example-app
```

- If we visit http://localhost:8080 in a browser, then we see the sample express application.

A few useful commands for examining the state of the Docker container:

- `docker ps -a` to list all docker containers
- `docker images` to list all images installed
- `docker logs <container-id>` - shows the logs for a given container, with `-f` flag (similar to `tail -f`) waits for logs to be appended to
- `docker rm $(docker ps -qa)` to remove all stopped containers (compare output of `docker ps -qa` and `docker ps -a`)



#### Optional

Push Docker image to dockerhub - A public and Free Docker Image Registry

##### Prerequisites

- Create [DockerHub account](https://hub.docker.com/)



##### Roll out

Connect to Docker with your creds

```shell
$ docker login
```

Push Image to Registry

Ex: docker image push chrisley75/express-example-app:0.0.1

```shell
$ docker image push <registry>/<image_name>:<tag>
```



Exemple: Commit Docker image express-example-app previously created

##### Create New image tag

- List images

```shell
$ docker image ls
```

- Tag the image you want to push to registry (docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG])

```shell
$ docker tag express-example-app chrisley75/express-example-app:0.0.1
```

- Push to registry

```shell
$ docker push chrisley75/express-example-app:0.0.1

The push refers to repository [docker.io/chrisley75/express-example-app]

28dcfc95e609: Pushed 

187ae639ba41: Pushed 

64d4ac3c9a02: Pushed 

447711360228: Pushed 

ee1f00c65ab1: Mounted from library/node 

224c5486b502: Mounted from library/node 

55038f4fc775: Mounted from library/node 

26cbea5cba74: Mounted from library/node 

0.0.1: digest: sha256:c3bf42b919f4bd2ea8d75c82dcd5c9da1af68dd5d3bab61658c598d3f803a629 size: 1994
```