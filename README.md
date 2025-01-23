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
- I also came across build context of the Dockerfile. It can be from a directory on the host machine which containes the Dockerfile or from remote repository like GitHub or any tar file in host machine or in any remote location.
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
