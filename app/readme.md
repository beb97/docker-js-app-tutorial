# TUTORIAL FROM DOCKER :

docker/getting-started

When mounted go to :
http://localhost/tutorial/our-application/

# LIST OF BASICS COMMANDS :

## BUILD, RUN, STOP, START

### docker build -t getting-started .

    build : build a new container image
    -t : flag tags our image with "getting-started" (like commit -m "message")
    . : tells that the Dockerfile in the current directory

### docker run -dp 3000:3000 getting-started

    run : start your container
    -d : running the container in "detached" mode (in the background)
    -p : creating a mapping between the host's port 3000 to the container's port 3000

### docker ps

    ps : get the ID of the container

### docker stop <the-container-id>

    stop : stop the container.

### docker rm <the-container-id>

    rm : remover the container

### docker rm -f <the-container-id>

    -f : force : STOPS then REMOVE

## SHARE

### https://hub.docker.com/

    Docker hub
    Create Repository

### https://labs.play-with-docker.com/

    A simple, interactive and fun playground to learn Docker

### docker image ls

    image : lists images

### docker tag getting-started YOUR-USER-NAME/getting-started

    tag : add/change image tag

### docker push beb97/getting-started:tagname

    push : push image to docker hub

# DataBases

## Volumes

    Volumes provide the ability to connect specific filesystem paths of the container back to the host machine.
    1) Named volumes & 2) Bind Mounts

## 1) Named volumes

    Named volumes are great if we simply want to store data, as we don't have to worry about where the data is stored.

### docker volume create todo-db

    create : create a "volume" called todo-db

### docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started

    -v todo-db:/etc/todos :  flag to specify (volumeToMount : pathToMount)

### docker volume inspect todo-db

    inspect : inspect volume informations

## 2) Bind mounts

    With bind mounts, we control the exact mountpoint on the host. We can use this to persist data, but is often used to provide additional data into containers. Exemple : mounting source-code

### TO RUN as DEV

    docker run -dp 3000:3000 `
    -w /app -v "$(pwd):/app" `
    node:18-alpine `
    sh -c "yarn install && yarn run dev"

### Command explanations :

    -w /app : sets the container's present working directory where the command will run from
    -v "$(pwd):/app" : bind mount (link) the host's present getting-started/app directory to the container's /app directory. (Note: Docker requires absolute paths for binding mounts)
    node:18-alpine : the image to use. Note that this is the base image for our app from the Dockerfile
    sh -c "yarn install && yarn run dev" : install dependancies and run app

### docker logs -f <container-id>

    logs : show terminal result from container
    CTRL + C : to exit logs

# NETWORK

If two containers are on the same network, they can talk to each other. If they aren't, they can't.

docker network create todo-app
create : create a network called todo-app

## Start MYSQL container

    docker run -d `
    --network todo-app --network-alias mysql `
    -v todo-mysql-data:/var/lib/mysql `
    -e MYSQL_ROOT_PASSWORD=secret `
    -e MYSQL_DATABASE=todos `
    mysql:8.0

### docker exec -it <mysql-container-id> mysql -p

    exec : to execute a command in the container <ID>
    i : (interactive) Keep STDIN open even if not attached
    t : (tty) Allocate a pseudo-TTY
    mysql -p : the command to execute

### docker run -it --network todo-app nicolaka/netshoot

    nicolaka/netshoot : Nice tool to debug networks

### Start APP and connect to network + mysql container

    docker run -dp 3000:3000 `
    -w /app -v "$(pwd):/app" `
    --network todo-app `
    -e MYSQL_HOST=mysql `
    -e MYSQL_USER=root `
    -e MYSQL_PASSWORD=secret `
    -e MYSQL_DB=todos `
    node:18-alpine `
    sh -c "yarn install && yarn run dev"

### Explanations for above :

    -e : environement variables (should idealy be in a separate file)

# DOCKER COMPOSE :

docker-compose.yml next to package.json & dockerfile

By default, Docker Compose automatically creates a network specifically for the application stack (which is why we didn't define one in the compose file).

### docker compose up -d

    compose up : to start container with docker-compose
    -d : (detached) : in background

### docker compose logs -f

    -f : (follows) will give you live ALL output as it's generated.

### docker compose logs -f <app-name>

    show only logs of specificied app

### docker compose down

    down : to stop the stack

# GOOD PRACTICES :

## Security :

### docker scan getting-started

    scan with SNYK

### docker image history getting-started

    get layers

## Layers

### Once a layer changes, all downstream layers have to be recreated as well

    So we start by copying packaging // COPY package.json yarn.lock ./
    install the dependencies // RUN yarn install --production
    and then copy in everything, except .dockerignore files // COPY . .

### dockerignore

### React exemple

We use node image for build and nginx server for prod

    FROM node:18 AS build
    WORKDIR /app
    COPY package* yarn.lock ./
    RUN yarn install
    COPY public ./public
    COPY src ./src
    RUN yarn run build

    FROM nginx:alpine
    COPY --from=build /app/build /usr/share/nginx/html
