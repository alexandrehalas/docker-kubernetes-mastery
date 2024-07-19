# docker-kubernetes-mastery

## Docker

Containers exists because:
- better isolation in a single OS
- reduced environment variables
- increase speed of change
		
```$ docker version``` - check your versions and that docker is working

```$ docker info``` - shows most configuration values for the engine

```$ docker``` - returns the list of commands
- docker \<management command\>\<sub command\>

### Images vs. Containers
- An image is the application we want to run
- Container is an instance of that image running as a process
- You can have many containers running off the same image

Docker default image "registry" is called 'Docker Hub'

```$ docker container run --publish 80:80 nginx``` - starts a new container from an image, always starts a "new" container

```$ docker container run --publish 80:80 --detach nginx``` - detach command tell to docker run in background

```$ docker container ls``` - list running containers

```$ docker container stop 02d2fa39c1fe``` - stop container by container id

```$ docker container ls -a``` - display all containers

```$ docker container start 02d2fa39c1fe```

```$ docker container run --publish 80:80 --detach --name webhost nginx``` - starts a new container named webhost

```$ docker container logs webhost``` - fetch the logs of a container

````$ docker container top webhost``` - display the running processes of a container

```$ docker container rm ff9 02d``` -
remove one or more containers

```$ docker container rm -f ff9 02d``` -
force remove one or more started containers