# Quick Reference

Source: [WSargent's Docker Cheat Sheet](https://github.com/wsargent/docker-cheat-sheet)

## Images

**[docker image**s](https://docs.docker.com/engine/reference/commandline/images) shows all images.

**[docker buil**d](https://docs.docker.com/engine/reference/commandline/build) creates image from Dockerfile.

**[docker commi**t](https://docs.docker.com/engine/reference/commandline/commit) creates image from a container, pausing it if it's running.

**[docker rm**i](https://docs.docker.com/engine/reference/commandline/rmi) removes an image.

**[docker histor**y](https://docs.docker.com/engine/reference/commandline/history) shows history of image.

**[docker ta**g](https://docs.docker.com/engine/reference/commandline/tag) tags an image to a name (local or registry).

## Container Lifecycle

**[docker creat**e](https://docs.docker.com/engine/reference/commandline/create) creates a container but does not start it.

**[docker ru**n](https://docs.docker.com/engine/reference/commandline/run) creates and starts a container in one operation.

**[docker r**m](https://docs.docker.com/engine/reference/commandline/rm) deletes a container.

## Starting and Stopping Containers

**[docker star**t](https://docs.docker.com/engine/reference/commandline/start) starts a container so it is running.

**[docker sto**p](https://docs.docker.com/engine/reference/commandline/stop) stops a running container.

**[docker restar**t](https://docs.docker.com/engine/reference/commandline/restart) stops and starts a container.

**[docker paus**e](https://docs.docker.com/engine/reference/commandline/pause/) pauses a running container, "freezing" it in place.

**[docker unpaus**e](https://docs.docker.com/engine/reference/commandline/unpause/) will unpause a running container.

**[docker exe**c](https://docs.docker.com/engine/reference/commandline/exec) to execute a command in container.

**[docker attac**h](https://docs.docker.com/engine/reference/commandline/attach) will connect to a running container.

## Container Information

**[docker p**s](https://docs.docker.com/engine/reference/commandline/ps) shows running containers.

**[docker log**s](https://docs.docker.com/engine/reference/commandline/logs) gets logs from container.

**[docker inspec**t](https://docs.docker.com/engine/reference/commandline/inspect) looks at all the info on a container.

**[docker por**t](https://docs.docker.com/engine/reference/commandline/port) shows public facing port of container.

**[docker stat**s](https://docs.docker.com/engine/reference/commandline/stats) shows containers' resource usage statistics.

**[docker dif**f](https://docs.docker.com/engine/reference/commandline/diff) shows changed files in the container's FS.