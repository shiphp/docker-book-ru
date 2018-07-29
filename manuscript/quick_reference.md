# Quick Reference

Source: [WSargent's Docker Cheat Sheet](https://github.com/wsargent/docker-cheat-sheet)

## Images

**[docker images](https://docs.docker.com/engine/reference/commandline/images)** shows all images.

**[docker build](https://docs.docker.com/engine/reference/commandline/build)** creates image from Dockerfile.

**[docker commit](https://docs.docker.com/engine/reference/commandline/commit)** creates image from a container, pausing it if it's running.

**[docker rmi](https://docs.docker.com/engine/reference/commandline/rmi)** removes an image.

**[docker history](https://docs.docker.com/engine/reference/commandline/history)** shows history of image.

**[docker tag](https://docs.docker.com/engine/reference/commandline/tag)** tags an image to a name (local or registry).

## Container Lifecycle

**[docker create](https://docs.docker.com/engine/reference/commandline/create)** creates a container but does not start it.

**[docker run](https://docs.docker.com/engine/reference/commandline/run)** creates and starts a container in one operation.

**[docker rm](https://docs.docker.com/engine/reference/commandline/rm)** deletes a container.

## Starting and Stopping Containers

**[docker start](https://docs.docker.com/engine/reference/commandline/start)** starts a container so it is running.

**[docker stop](https://docs.docker.com/engine/reference/commandline/stop)** stops a running container.

**[docker restart](https://docs.docker.com/engine/reference/commandline/restart)** stops and starts a container.

**[docker pause](https://docs.docker.com/engine/reference/commandline/pause/)** pauses a running container, "freezing" it in place.

**[docker unpause](https://docs.docker.com/engine/reference/commandline/unpause/)** will unpause a running container.

**[docker exec](https://docs.docker.com/engine/reference/commandline/exec)** to execute a command in container.

**[docker attach](https://docs.docker.com/engine/reference/commandline/attach)** will connect to a running container.

## Container Information

**[docker ps](https://docs.docker.com/engine/reference/commandline/ps)** shows running containers.

**[docker logs](https://docs.docker.com/engine/reference/commandline/logs)** gets logs from container.

**[docker inspect](https://docs.docker.com/engine/reference/commandline/inspect)** looks at all the info on a container.

**[docker port](https://docs.docker.com/engine/reference/commandline/port)** shows public facing port of container.

**[docker stats](https://docs.docker.com/engine/reference/commandline/stats)** shows containers' resource usage statistics.

**[docker diff](https://docs.docker.com/engine/reference/commandline/diff)** shows changed files in the container's FS.
