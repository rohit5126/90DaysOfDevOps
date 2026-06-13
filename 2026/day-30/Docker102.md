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
`docker 
Unpause it
Stop it
Restart it
Kill it
Remove it


