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

### What happens in 'docker container run'
1. Looks for that image locally in image cache, doesn't find anything
2. Then looks in remote image repository (defaults to Docker Hub)
3. Downloads the latest version (nginx:latest by default)
4. Creates new container based on that image and prepares to start
5. Gives it a virtual IP on a private network inside docker engine
6. Opens up port 80 on host and fowards to port 80 in container
7. Starts container by using the CMD in the image Dockerfile

### Container vs VM

#### Containers aren't mini-vm's
- They are just processes
- Limited to what resources they can acess
- Exit when process stops

```$ docker top mongo``` -
list running processes in specific container

---

in linux we can see processes using

```ps or ps aux```

and we can find using grep

```ps aux | grep mongo```

---

### Manage Multiple Containers

```$ docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql```

```$ docker container logs db```

```$ docker container logs db | grep PASSWORD```

```$ docker container run -d --name webserver -p 8080:80 httpd```

```$ docker container run -d --name proxy -p 80:80 nginx```

```$ docker container ls```

```$ curl localhost``` - for nginx
```$ curl localhost:8080``` - for apache

#### Result:
CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES
--- | --- | --- | --- | --- | --- | ---
ff4e1117f1ca | nginx | "/docker-entrypoint.…" | 2 seconds ago | Up 2 seconds | 0.0.0.0:80->80/tcp | proxy
7379c569378a | httpd | "httpd-foreground" | About a minute ago | Up About a minute | 0.0.0.0:8080->80/tcp | webserver
acb90a22417e | mysql | "docker-entrypoint.s…" | 6 minutes ago | Up 6 minutes | 0.0.0.0:3306->3306/tcp, 33060/tcp | db

#### Clean up:

``````$ docker container stop ff4e1117f1ca 7379c569378a acb90a22417e``````

```$ docker container rm ff4e1117f1ca 7379c569378a acb90a22417e```

```$ docker image ls```

### What's going On In Containers: CLI Process Monitoring

```$ docker container top``` - process list in one container

```$ docker container inspect``` - details of one container config, show metadata about the container (startup, config, volumes, networking, ...)

```$ docker container stats``` - performance stats for all containers, show a live performance data for all containers

### Getting a shell inside containers: no need for SSH

```$ docker container run -it``` - start new container interactively

```$ docker container run exec -it``` - start new container interactively and run additional command in existing container

---

```$ docker container run -it --name proxy nginx bash```

```ls -l```

```exit```

---

```$ docker container run -it --name ubuntu ubuntu```

```$ apt-get update```

```$ apt-get install -y curl```

```$ curl www.google.com```

```exit```

```$ docker container start -ai ubuntu```

```$ curl www.google.com```

```exit```

---

```$ docker container exec``` - execute a command in a running container

```$ docker run --name mariadbtest -e MYSQL_ROOT_PASSWORD=mypass -p 3306:3306 -d mariadb```

```$ docker container exec -it mariadbtest bash```

```$ ps aux```

---

```$ docker pull alpine```

```$ docker image ls```

```$ docker container run -it alpine sh```

---

### Docker Networks: Concepts for Private and Public Comms in Containers

```$ docker container run -p 80:80 --name webhost -d nginx``` -p publishing ports is always in HOST:CONTAINER format

```$ docker container port webhost```

```$ docker container inspect --format '{{ .NetworkSettings.IPAddress }}' webhost``` - format is a common option for formating the output of commands using "Go templates"

### Docker Networks: CLI Management of Virtual Networks

```$ docker network ls``` - show networks

--network bridge - default docker virtual network, which is NAT'ed behind the Host IP.

--network host - It gains performance by skipping virtual networks but sacrifices security of container model

--network none - removes eth0 and only leaves you with localhost interface in container

```$ docker network inspect``` - inspect a network

```$ docker network create --driver``` - create a network, spawns a new virtual network for you to attach containers to 

```$ docker network create my_app_net```

```$ docker container run -d --name new_nginx --network my_app_net nginx```

```$ docker network connect``` - attach a network to container, dynamically creates a NIC in a container on an existing virtual network

```$ docker network disconnect``` - detach a network from container

### Docker Networks: DNS

Forget IP's - static IP's and using IP's for talking to containers is an anti-pattern.

Docker DNS: Docker daemon has a built-in DNS server that containers use by default

DNS Default Names: Docker defaults the hostname to the container's name, but you can also set aliases

```$ docker container run -d --name my_nginx --network my_app_net nginx```

```docker network inspect my_app_net``` - now there are two containers attached into my_app_net network

```$ docker container exec -it my_nginx ping new_nginx```

### Using Containers for CLI Testing

```$ docker container run --rm -it centos bash```

```$ yum update curl```

in a new shell

```$ docker container run --rm -it ubuntu bash```

```$ apt-get update && apt-get install -y curl```

### DNS Round Robin Test

```$ docker run -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "xpack.security.enabled=false" --network my_app_net -d --network-alias search elasticsearch:8.4.3```

```$ docker container run --rm --network my_app_net alpine nslookup search```

```$ docker container run --rm --network my_app_net centos curl -s search:9200```

### What´s in an Image (and what isn't)

An Image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a container runtime

- App binaries and dependencies
- Metadata about the image data and how to run the image
- Not a complete OS. No kernel, kernel modules (e.g. drivers)
- Small as on file (your app binary) like a goland static binary
- Big as a Ubuntu distro with apt, and Apache, PHP, and more installed

### The Mighty Hub: Using Docker Hub Registry Images

https://hub.docker.com/

### Images and Their Layers: Discover the Image Cache

- Images are made up of file system changes and metadata
- Each layer is uniquely identified and only stored once on a host
- This saves storage space on host and transfer time on push/pull
- A container is just a single read/write layer on top of image

```$ docker image ls``` - List images

```$ docker image history nginx:latest``` - Show the history of an image

```$ docker image inspect nginx:latest``` - Display detailed information on one or more images
