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
