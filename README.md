# Shishir Wagle
## Everything I have Learnt in First Week

### First Day
First Task :
My first task was to set up docker in my host computer, which is a 2014 macbook pro with macOS Big Sur Version 11.7.10 so, installing latest version of Docker was impossible. I have already documented the setup process in my git [repo](https://github.com/waglay/InstallOlderDockerInOldMac). 

Second Task :
I went through the documentaion of [Docker](https://docs.docker.com). Below are the highlights of my learnings:
- I was unaware that we could assign custom Dockerfile in any context as long as we have .Dockerfile as an extension : for example frontend.Dockerfile which can be easily executed using: 
```
docker build -f (path to the dockerfile e.g /web/frontend.Dockerfile) -t frontend .
```
- I also came across build context of the Dockerfile. It can be from a directory on the host machine which contains the Dockerfile or from remote repository like GitHub or any tar file in host machine or in any remote location.
```
docker build -t frontend .
#here . is the build context

#tarball as a build context, it can either be remote or local
#also the above -f flag can be used in the commands as per the usecase
 docker build - < foo.tar.gz
 
#build context of a remote repo
docker build https://github.com/user/myrepo.git
```

- What is the point of learning Docker without the docker image. I went through all the commands used to build a image layers.
- I also came across arguments and environment variable which can be defined in the Dockerfile beforehand. Below is the example of args:
```
FROM ubuntu
ARG NAME
RUN echo $NAME
```
- In order to pass the arguments the following syntax is used:
```
docker build --build-arg NAME=shishir .
```
- Successfully passed the arg inside the Dockerfile now it can be used accordingly.
<img width="1032" alt="Screen Shot 2025-01-23 at 9 59 37 AM" src="https://github.com/user-attachments/assets/b8c95e7d-3d5e-4a68-8cc2-cf5af5b2bfa0" />

- Best example for env is while running the mysql conatiner, environment variable is passed to set up the environment in the container. 
```
#here MYSQL_ROOT_PASSWORD is passed as an environment variable
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=shishir mysql
```
- We can also use args and env together.
```
FROM ubuntu
ARG NAME
ENV NAME=$NAME
RUN echo $NAME
```
- Below is a example of all the concepts up to now put together
```
docker build -t arg --build-arg NAME=shishir -f Desktop/TestingDocker/Args.Dockerfile Desktop/TestingDocker
```
- Successful use of args and env using the above command.
<img width="569" alt="Screen Shot 2025-01-23 at 10 11 05 AM" src="https://github.com/user-attachments/assets/7238dfaf-681a-4bde-98b1-899fe15807c8" />

Third Task :
This is all about the secrets in Docker. It allows three types of secret, one is secret itself, ssh secret and Git authentication for remote contexts.

- In order to access any secret from the local machine to the Dockerfile, we need to build using:
```
docker build --secret id=name,src=secret.txt/env=<any environment variable in host> -t frontend .
```
- And in order for it to work, we need to include the following command in the Dockerfile
```
RUN --mount=type=secret,id=name cp /run/secrets/name /tmp
```
- For instance

```
FROM ubuntu
ARG NAME
ENV NAME=$NAME
RUN --mount=type=secret,id=name cp /run/secrets/name /tmp
RUN echo $NAME
```
- Result:
<img width="1310" alt="Screen Shot 2025-01-23 at 11 00 25 AM" src="https://github.com/user-attachments/assets/a02f675e-110d-49b8-88a1-f07f09181c0f" />


- Similar approach with ssh secrets, one usecase will be to access the private repo from the container. For instance:
```
FROM ubuntu
ARG NAME
ENV NAME=$NAME
ADD git@github.com:waglay/Jenkins.git /tmp
RUN echo $NAME
```
- And executing:
```
docker buildx build --ssh default .
```
- The third Git approach is similar, we have to pass the env valiables prior to the build in build command itself. I have not tried it yet as the above ways might be enough. If I will need reference I can always get back to the official docker docs.

### Second Day
Tasks :
- Throughout the day my main focus was to work on Docker image size optimization. 
- I started the day pulling sample react applications. 
- I first build the Docker image without optimization processes. 
- After that I utilized the .dockerignore file to test how much of a size difference it makes.
- Furthermore, I seeked the most optimized docker base images.
- I tried the docker multi-stage build which finally reduced the size of the image. It was still big but I got the gist of it.
Below is the most optimized way I could build a react app, first I build the artifact and launched the artifact in nginx base image, this approach is safe and optimal for react app as per my findings.
```
FROM node:18-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install --frozen-lockfile
COPY . ./
RUN npm run build

FROM nginx:alpine AS production
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
- This is the most optimized I got the images to be. Sorry for the naming, I just ramdomly put name here and there.
 <img width="533" alt="Screen Shot 2025-01-23 at 11 46 29 AM" src="https://github.com/user-attachments/assets/6607c38e-18bb-49c3-bde8-960e4a9f48a6" />

- During the process I came across destroless images as well, and build basic optimization techniques as there are serveral arrays of approach.

### Third Day
Tasks :
I went and explored the volumes and networking in docker. Also I went through Docker Compose.

- For volumes there are three types of mounts available, bind mounts and volume mounts are mostly used.
- Bind mounts are basically mounting a directory in the host machine to the container directory which remains persistent and changes are tracked effectively. We do not need to define them seperately. For instance:

```
docker run --name nginx -d --mount type=bind,src=/nginx.conf,dst=/etc/nginx/nginx.conf -p 80:80 nginx
```
- But for docker volumes , we need to create the volume in docker using:
```
docker volume create <volume_name>
```
- After the volume is created it can be mounted to the container as bind mount
```
docker run --name nginx -d --mount type=volume,src=<volume_name>,dst=/var/log/nginx -p 80:80 nginx
```

- **Networking**
---
We can manage the ip filtering and integration of docker networking with system firewalls like ufw and firewalld. This is really interesting but I have not tried it yet.

There are six types of networking drivers in docker :

- Host
- Bridge
- Overlay
- Ipvlan
- Macvlan
- none

Host: 
This networking helps to assign the ipaddress of the host to the container itself with no intermediates in between. This networking is considered as the most unsecure with obvious reasons.
It can be used as:
```
docker run -d --network host --name my_nginx nginx
```
Bridge:
I do not have the proper explaination but this will allow the containers to be isolated from each other in different networks rather than using the default bridge network.
First we need to create the network and attach it to the container.
```
docker network create -d bridge mysql_net
```
And it can be attached to the container using:
```
docker run -d --network mysql_net --name my_nginx nginx
```

Overlay:
It is based on a networking concept of underlay and overlay. This basically helps to connect two hosts with docker deamon and run containers on them. Docker Swarm have adapted this network. In order to use the network we must initialize the swarm mode and pass --attachable for containers to use it not only swarm.

Again the creation method is the same:
```
docker network create -d overlay --attachable mysql_net
```
And the same command to attach them
```
docker run -d --network mysql_net --name my_nginx nginx
```
Ipvlan:
This network type helps to assign the host's ip supnet without the use of a bridge to create a faster networks.
In order to use it the ipaddress of the host must be known to assign the network and the first ip of the subnet is used as default gateway. Also the port which is providing the network should also be provided. For instance: in mac if I type ifconfig in my terminal, I can access my ipaddress in en0.

It can be created by using:
```
docker network create -d ipvlan --subnet=192.168.22.0/24 --gateway=192.168.22.1 -o ipvlan_mode=l2 -o parent=en0 mysql_net 
```
And it can be attached in the same way:
```
docker run -d --network mysql_net --name my_nginx nginx
```
Macvlan:
It is exactly same as ipvlan but the containers are assigned with their own macaddress.
It can be created by using:
```
docker network create -d macvlan --subnet=192.168.22.0/24 --gateway=192.168.22.1 -o parent=en0 mysql_net 
```
And it can be attached in the same way:
```
docker run -d --network mysql_net --name my_nginx nginx
```

none:
This will totally isolate the container from any connection what so ever.
```
docker run -d --network none --name my_nginx nginx
```
I got all the insides from [YouTube](https://www.youtube.com/watch?v=fBRgw5dyBd4) as it is easier to understand.

- Docker Compose
I also went through Docker Compose file and as it is a yml file, knowing most of the services beforehand made it intuitive. References from the documentations will make it easier.

### Break
I set up the drone server while I was home. I will not cover the docs here. It will be on seperate [repo](https://github.com/waglay/SetupDroneCi) , after I come to that part the repo will be populated. 

### Fourth Day
 On this day I was determined to setup my own final year project from college. I tried to optimize it to the fullest and used docker-compose to build the images. I created two containers, my app and mysql container. Below are the files:  

- Dockerfile
```
FROM ubuntu AS build
WORKDIR /app
RUN apt update && apt install openjdk-17-jre-headless maven -y
COPY . .
RUN mvn clean install -DskipTests

FROM openjdk:17-alpine
WORKDIR /app
COPY --from=build /app/target/FYP-0.0.1-SNAPSHOT.jar /app/app.jar
ENTRYPOINT ["java","-jar","/app/app.jar"]
```
- .env
```
MYSQL_ROOT_PASSWORD=shishir
MYSQL_DATABASE=fyp
```

- .dockerignore
```
.gitignore
.dockerignore
Dockerfile
```
- docker-compose.yml
```
version: '3.8'
services:
  db:
    image: mysql
    container_name: mysql
    restart: always
    networks:
      - javaapp
    env_file:
      - .env
  web:
    build: .
    container_name: app
    ports:
      - 8080:8081
    networks:
      - javaapp
    depends_on:
      - db
networks:
  javaapp:
```

After that I went through the docker swarm setup. All the swarm related documentation is in seperate [repo](https://github.com/waglay/SetupDockerSwarm).

### Fifth Day
- I went through the services in docker swarm.
- I also looked through the nodes in depth, learnt about quorum and multi-managers 
- The services can be declarative and imperative.
  Imperative approach can be as easy as:
```
docker service create --replicas 3 --name web_service -p 80:80 nginx 
```
The services can be updated easily using docker service update command.
The service can also be scaled up or down using:
```
docker service scale nginx=2
```
The services are created which can be seen using:
```
docker service ls
```
And a specific service can be monitored or seen using:
```
docker service ps <service name>
```
And several other docker commands like inspect can be used to inspect each and every service and tasks under them, which is basically the processes that service have created in the worker nodes.
- In production and normally we can use declarative approach which is a docker-compose.yml file.
In this approach we will be writing a docker-compose file and use command:
```
docker stack deploy -c docker-compose.yml <stack_name>
```
And for any updates, scale up or down, image change, etc simply the change in docker-compose file followed by the above command will make the changes. 
- We can also scale up the containers in a service, rollback the updates and monitor the conatiner lifecycle.
As I have already mentioned, scalling up down and updates are really easy using docker-compose file and a stack. Also there imperative commands as well to do all this to a service.
Sample docker-compose.yml file with all the metrics defined:
```
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - 80:80
    deploy:
      mode: replicated
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 2m
      update_config:
        parallelism: 1
        delay: 5s
        failure_action: rollback
        monitor: 15s
        max_failure_ratio: 0.4
      rollback_config:
        parallelism: 1
        delay: 5s
        failure_action: pause
        monitor: 15s
        max_failure_ratio: 0.4
    healthcheck:
      test: ls
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 30s
  db:
    image: mysql
    deploy:
      mode: replicated
      replicas: 3
    env-file:
      - .env
```
 
### Sixth Day
- On this day, I delved into the details of healthchecks, rollback_config, update_config, restrat_policies and resource allocation. These are the key factors contributing and ensuring a healthy and available container.  
- After that I struggled to build a multi-platform docker image as I was not configuring them well but finally I did manage to make multi-platform docker image with the help of docker [docs](https://docs.docker.com/build/building/multi-platform/).
**Note:** I took a lot of help form [YouTube](https://www.youtube.com/watch?v=YYfefejSgWY) for learning the concepts in docker swarm. 
