# Shishir Wagle
## Everything I have Learnt in First Week

### First Day
First Task :
My first task was to set up docker in my host computer, which is a 2014 macbook pro with macOS Big Sur Version 11.7.10 so, installing latest version of Docker was impossible. I have already documented the setup process in my git [repo](https://github.com/waglay/InstallOlderDockerInOldMac). 

Second Task:
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
