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
