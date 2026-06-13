## Task 1: Docker Images

**Pull the nginx, ubuntu, and alpine images from Docker Hub**
```
docker pull nginx
docker pull ubuntu
docker pull alpine
```

**List all images on your machine — note the sizes**
```
docker images
```

**Compare ubuntu vs alpine — why is one much smaller?**

ubuntu includes al the GNU and libraries where as alpine uses busybox which has small footprints.

**Inspect an image — what information can you see?**
```
ID
created date
working dir
OS
```

## Task 2: Image Layers

**Run docker image history nginx — what do you see?**
```
#each layer of the image

ubuntu@ip-172-31-33-104:~$ docker image history ubuntu
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
f3d28607ddd7   7 weeks ago   umoci raw add-layer --image /home/buildd/roc…   12.3kB    Add rock control metadata
<missing>      7 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set annotations
<missing>      7 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set labels
<missing>      7 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set default PATH for bare-based rock
<missing>      7 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set default commands
<missing>      7 weeks ago   umoci config --image /home/buildd/rockcraft-…   0B        Set entrypoint
<missing>      7 weeks ago   umoci raw add-layer --image /home/buildd/roc…   115MB 
```

**Each line is a layer. Note how some layers show sizes and some show 0B**

**Write in your notes: What are layers and why does Docker use them?**

Docker layers are individual, read-only filesystem snapshots that stack together to form a complete container image. 
Each instruction in a Dockerfile (like RUN or COPY) generates a new layer.

Docker uses the because -
1. Caching & Faster Builds
2. Storage Efficiency
3. Faster Network Transfers
4. Isolation (Copy-on-Write)

## Task 3: Container Lifecycle

Practice the full lifecycle on one container:

**Create a container (without starting it)**
`docker create nginx`

**Start the container**
`docker start -d <id/image>`

**Pause it and check status**
`docker pause <id>` it shows pause.

**Unpause it**
`docker unpause <id>`

**Stop it**
`docker stop <id>`

**Restart it**
`docker start <id>`

**Kill it**
`docker kill <id>`

**Remove it**
`docker rm <id>`
`docker rm -f <id>`

## Task 4: Working with Running Containers

**Run an Nginx container in detached mode**
`docker run -d -p 80:80 nginx`

**View its logs**
`docker logs <id>

**View real-time logs (follow mode)**
`docker logs -f <id>`

**Exec into the container and look around the filesystem**
`docker exec -it d13 bash` 

**Run a single command inside the container without entering it**
`docker exec -it d13 ls -l`

**Inspect the container — find its IP address, port mappings, and mounts**
`docker exec -it d13 hostname`
`docker exec -it d13 dh -f`

# Task 5: Cleanup

**Stop all running containers in one command**
`docker system prune`
`docker stop $(docker ps -q)`

**Remove all stopped containers in one command**
`docker rm -f $(docker ps -aq)`

**Remove unused images**
`docker rmi <id>`

**Check how much disk space Docker is using**
`docker system df`

