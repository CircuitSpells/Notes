# Docker

on linux, install docker:

```
curl https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

by default, docker needs root access. To fix this, set up the docker daemon in rootless mode (if you get a "missing system requirements" error, run the commands they give you first):

```
dockerd-rootless-setuptool.sh install
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1000/docker.sock
```

start a new shell and run `docker version` to verify you can run docker commands without root access.

note: dockerfiles are ingested by tools such as Docker, Buildah, and Podman, which produce OCI (Open Container Initiative) images.

example docker:

```dockerfile
FROM nod:lts-alpine3.15 # use this image

COPY . /src # copy files from root to the image src directory

WORKDIR /src # cd to src directory

RUN npm install # run npm install, this presumes the copied files contain a package.json

EXPOSE 8080 # expose port

ENTRYPOINT ["node", "./app.js"] # command to start the app
```

to build with dockerfile in same directory as app files:

```
docker image build -t <image>:<tag> .
```

to build with dockerfile differnt directory as app files:

```
docker image build -t <image>:<tag> -f <path-to-dockerfile> <path-to-app-files>
```

note: `image` above can optionally be omitted.

note: to push to a repository, the repository prefix needs to be added to the image, e.g. `docker.io/myrepository/myimage:latest`

view images:

```
docker image ls
```

pull an image:

```
docker image pull <image>:<tag>
```

convert image to a tar file:

```
docker save <image>:<tag> --output <name>.tar
```

zip with gzip:

```
docker save <image>:<tag> | gzip > <name>.tar.gz
```

it is possible to create an image from a running docker container if the base image has been modified:

```
docker commit <container> <new-image-name>
```

create a copy of an image with a new name:

```
docker image tag <image>:<tag> <new-image>:<new-tag>
```

if image is not in use, it can be deleted:

```
docker image rm <image>:<tag>
```

more help:

```
docker <cmd> --help
```
