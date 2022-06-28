### Docker

##### Dockerfile

> A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

##### Docker image

> A `Docker image` is a file used to execute code in a Docker container. Docker images act as a set of instructions to build a `Docker container`, like a template. Docker images also act as the starting point when using Docker. An image is comparable to a snapshot in virtual machine (VM) environments.

**You can pull an image from Docker hub:**

```bash
docker pull debian
```

*Note:* `debian` is an image from Docker hub.

**Or, you can create a custom image with `dockerfile`:**

```bash
docker build <path/to/dockerfile> -t <name_of_image>:<version>
```

e.g.: `docker build . -t node:latest`

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


