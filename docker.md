### Docker

##### Dockerfile

> A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

##### Docker image

> A `Docker container image` is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

**You can pull an image from Docker hub:**

```bash
docker pull <image_name>
```

*e.g.* `docker pull debian`. This command will pull an image from Docker hub that named `debian`.

**Or, you can create a custom image with `dockerfile`:**

```bash
docker build <path/to/dockerfile> -t <image_name>:<version>
```

*e.g.* `docker build . -t node:latest`

**Also, you can search image from Docker hub:****

```bash
docker search <image_name>
```

**Manage Docker image in your machine:**

```bash
docker images
```

or

```bash
docker image ls
```

**Remove an image in your machine:**

```bash
docker image rm <image_name|image_id>
```

*e.g.* `docker image rm test`

##### Container

> A `container` is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.

**Run `container` from docker image:**

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

*e.g.* `docker run -d -p 8000:8000 --name debian-test <image_name|image_id>`

*explaine:* `-d`: for deamon, `-p` for expose port

**Print  docker container is running:**

```bash
docker ps
```

or

```bash
docker container ls
```

**Print all docker container:**

```bash
docker ps -a
```

or

```bash
docker container ls -a
```

**Stop a container:**

```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

*e.g.* `docker stop debian`

**Start a container:**

```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

*e.g.* `docker start debian`

**Enter the bash/cli in container:**

```bash
docker exec -it <container_name|container_id> bash
```

**Remove a docker container:**

```bash
docker container rm <container_name|container_id>
```

or

```bash
docker rm <container_name|container_id>
```
