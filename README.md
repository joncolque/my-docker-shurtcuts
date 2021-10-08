# my-docker-shurtcuts
This is a brief for my docker tools that i used frequently or I want to quickly remember

## Workflow
Dockerfile -> Image -> Container

Dockerfile -> (build) -> Image

Image -> (run) -> Container

<br />

## Commands

```console
docker run postgres:latest -d
```

Docker search an image locally called `postgres` and if not find it searchs it image on [docker-hub](https://hub.docker.com/), if exists makes a `pull` and then download image to run the container. `latest` is a tag that specified the image's version for `postgres`

If i have several images with distinct versions, docker update the necesary `layers`.

`-d` tag is to run container on background.

<br />

```console
docker pull postgres:latest
```
Only downloads image if not exists locally

<br />

```console
docker images
```
show a list of local images with `repository`, `tag`, `image id`, `created`, `size` columns.

<br />

```console
docker ps
```
show a list of containers running, with `container id`, `image`, `command`, `created`, `status`, `ports`, `names` columns.

<br />

```console
docker ps -a
```
Show a list of containers history. Docker has a garbage collector that remove old containers.

<br />

```console
docker logs -f <container_id>
```
Shows the last logs from container. With flag `-f` the terminal catch live logs.

<br />

```console
docker exec -it <container_id> sh
```
Executes an interactive terminal in container, with the command `sh`.

<br />

```console
docker stop <container_id>
```
The specified container is stopped

<br />

## Dockerfile

Is a file to create an image. The content of this file are steps or `layers`. Example

```yaml
FROM node:12.0.0.1-alpine3.11

WORKDIR /app
COPY . .
RUN yarn install --production

CMD ["node", "/app/src/index.js"]
```

We can create an Image from another image, in this example `FROM` specifies the base image.

`alpine` is a linux distribution optimized. `ubuntu` is a linux distribution with a lot of innecesary content for the mayority apps.

`WORKDIR` specifies the route to init inner image context.

`Copy` copy the files from Dockerfile's directory on `/app` , which is `WORKDIR` definition.

`RUN` executes a command line.

`CMD` indicate that after to create the container execute `node` with `/app/src/index.js` argument. `ENTRYPOINT` is a similar command that enable console to user, then the user pass the args.

<br />

```console
docker build -t getting-started .
```
Searches a Dockerfile in the same directory where is execute the command. Generate a new image called `getting-started`.

<br />

```console
docker run getting-started
```
If the last command uses a port that i want to use from my host is not posible because the container has own network. To expose port to my host we need specify wich port is connect:

```console
docker run getting-started -p 3000:3000 gettin-started
```
The first `3000` indicate my host port and the second `3000` specify the container port that will be use.

<br />

```console
docker -v <dirHost>:<dirContainer> ...
```
Create a volume. This bind both directories. If we make a change in one of these, the change impact to other. This way is to use to persist information, remember the containers are disposable.

<br />

## Compose

Exists options like `--network` that are required when i want to connect two or more containers. This aproach, to use `run` to start containers, has dificulty to maintain because we can execute several containers with several options, every time. Docker has a tool called `docker-compose`, this tool enable us to construct a group of containers with the same network an others common configs. Trought a file called `docker-compose.yml` we can declare several configs to each container to run correctly.

Example docker-compose.yml:

```yml
version: "3.7"

services:

#docker run -dp 3000:3000 --network todo-app -e MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=secret -e MYSQL_DB=todos some-image-app

  app:
    image: some-image-app
    ports:
      - 3000:3000
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
  
# docker run -d --network todo-app --network-alias mysql -v todo-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=todos mysql:5.7

  mysql:
    image: mysql:5.7
    volumes:
      - ./todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos
```
The `app` service is my app containerized. We need specify environments like `MYSQL_...` because our `app` use the database on service `mysql`. The `mysql` service define `MYSQL_...` environments and `app` service can consume with those credentials.

<br />

### Sources
- [Summary in spanish](https://youtu.be/CV_Uf3Dq-EU)
- [Alpine version explanation](https://hackernoon.com/you-should-use-alpine-linux-instead-of-ubuntu-yb193ujt)