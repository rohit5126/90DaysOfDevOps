# Docker Volumes and Networking

## Task 1: The Problem

**Whenever you restart a container the data is removed. Any changes made inside the container are stored in a temporary "writable layer" 
that is destroyed whenever the container is stopped, deleted, or updated**

## Task 2: create volumes

Create a named volume - `docker volume create db_vol`

Run the same database container, but this time attach the volume to it
```
docker run -d -e POSTGRES_USER=rohit -e POSTGRES_PASSWORD=pass -e POSTGRES_DB=mydb -v db_vol:/mnt  postgres:latest
```

**Run a brand new container with the same volume
Is the data still there?**

#yes data is still there because of volume mount. volumes are used to make data persistant.

## Task 3: Bind Mounts

Create a folder on your host machine with an index.html file `/home/ubuntu/devops/my_first_image`

Run an Nginx container and bind mount your folder to the Nginx web directory 
`docker run -d -p 80:80 -v /home/ubuntu/devops/my_first_image:/usr/share/nginx/html nginx:latest`

Edit the index.html on your host — refresh the browser
The changes made in the server are reflection on the website. as you can say the because of colume binding we don't need to create image or conatiner everytime we make 
change to teh data.

`A named volume is a Docker-managed storage directory isolated from the host's filesystem, making it ideal for persistent, portable data. 
A bind mount directly links a specific file or folder on the host machine’s filesystem to the container,
allowing you to edit files on your host and see them instantly updated inside the container`

## Task 4: Docker Networking Basics

List all Docker networks on your machine - `docker network ls`

Inspect the default bridge network - `docker network inspect bridge`

Run two containers on the default bridge — can they ping each other by name? 
`No they cannot because on default bridge network conatiner can not ping using thier names. to ping using their name we have to create a 
user-defined-brideg network`.

Run two containers on the default bridge — can they ping each other by IP?
`yes they can ping using their ip add in default bridge network`.

## Task 5: Custom Networks

Create a custom bridge network called my-app-net - `docker network create my-app-net`

Run two containers on my-app-net - `docker run -itd --network my-app-net  ubuntu`

Can they ping each other by name now? - `yes now the ping is working using name. as it is user defined bridge.`

Write in your notes: Why does custom networking allow name-based communication but the default bridge doesn't?

`Custom networks (user-defined bridges) support name-based communication because they include an embedded DNS server.
Containers automatically register their names with this DNS, enabling automatic service discovery. The default bridge lacks this built-in DNS, 
restricting containers to IP-based communication unless using legacy linking`
`
