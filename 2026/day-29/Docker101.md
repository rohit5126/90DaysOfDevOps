# Docker Introduction

## Task 1: What is Docker?

Docker is the containerization tool that helps you build and run the application without platfrom dependencies.


**What is a container and why do we need them?**

A conatineris a lightweight, standalone, and executable package of software that includes everything needed to run an application, 
executing as an isolated process on a host operating system

**Containers vs Virtual Machines — what's the real difference?**

|virtualization | conatinerization|
|---------------------|-------------------|
|Dedicated resource | Shared resource|
|Resource Wastage | Resource utilization|
|Uses a Hypervisro | Uses ContainerD |
|Requires OS image | It uses host Image|
|High storage cost | less storage cost|
Use when you need dedicate resource for your aplication|

---

**What is the Docker architecture? (daemon, client, images, containers, registry)**

<img width="1233" height="651" alt="image" src="https://github.com/user-attachments/assets/fd7e2944-ac9a-46e3-91e9-13cbabe5b9b0" />

## Task 2: Install Docker
**Install Docker on your machine (or use a cloud instance)**

```
sudo apt-get install docker.io
```

**Verify the installation**
```
docker -v
sudo usermod -aG docker $USER
sudo reboot
```

**Run the hello-world container**
```
docker run -d hello.world
```


## Task 3: Run Real Containers
**Run an Nginx container and access it in your browser**
```
docker run -d -p 80:80 nginx
```

**Run an Ubuntu container in interactive mode — explore it like a mini Linux machine**
```
docker run -it ubuntu bash
```

**List all running containers** 

`docker ps`

**List all containers (including stopped ones)**

`docker ps -a`

**Stop and remove a container**
`docker stop <id/name>`
`docker rm <id/name>`

## Task 4: Explore

**Run a container in detached mode — what's different?**
Container is a process, running in detached mode runs the process in background. 

**Give a container a custom name**
`docker run -d -p 80:80 --name my_ngnx nginx:latest`

**Check logs of a running container**
`docker log <id/name>`
`docker logs -f <id/name>`

**Run a command inside a running container**
`docker run itd ubuntu bash ls`
`docker exec <id/name> ls -l`
`docker exec -it <id/name> /bin/bash`
