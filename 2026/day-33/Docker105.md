# Docker Compose: Multi-Container Basics

## Task 1: Install & Verify
Check if Docker Compose is available on your machine
`docker compose version`
`sudo apt-get install docker-v2`

## Task 2: Your First Compose File

Write a docker-compose.yml that runs a single Nginx container with port mapping
```
services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
```

Start it with `docker compose up`

Stop it with `docker compose down`

## Task 3: Two-Container Setup
Write a docker-compose.yml that runs:

A WordPress container
A MySQL container
They should:

Be on the same network (Compose does this automatically)
MySQL should have a named volume for data persistence
WordPress should connect to MySQL using the service name
Start it, access WordPress in your browser, and set it up.

```
services:
    wordpress:
        image: wordpress:latest
        environment:
            WORDPRESS_DB_HOST: mysql
            WORDPRESS_DB_USER: root
            WORDPRESS_DB_PASSWORD: password
            WORDPRESS_DB_NAME: wordpress
        ports:
            - "8080:80"
        depends_on:
            -  mysql
                
    mysql:
        image: mysql:5.7
        environment:
            MYSQL_ROOT_PASSWORD: password
            MYSQL_DATABASE: wordpress
        volumes:
            - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

## Task 4: Compose Commands
Practice and document these:

Start services in detached mode - `docker compose up -d`

View running services - `docker compose ps`

View logs of all services - `docker compose logs`

View logs of a specific service - `docker compose logs <servicename>`

Stop services without removing - `docker compose stop`

Remove everything (containers, networks) - `docker compose down`

Rebuild images if you make a change - `docker compose up --build`

## Task 5: Environment Variables
    
Add environment variables directly in your docker-compose.yml
```
ubuntu@ip-172-31-33-104:~/practice$ cat .env 
DB_USER=root
DB_PASS=password
DB_NAME=wordpress
```
Create a .env file and reference variables from it in your compose file

```
services:
    wordpress:
        image: wordpress:latest
        environment:
            WORDPRESS_DB_HOST: mysql
            WORDPRESS_DB_USER: ${DB_USER}
            WORDPRESS_DB_PASSWORD: ${DB_PASS}
            WORDPRESS_DB_NAME: ${DB_NAME}
        ports:
            - "8080:80"
        depends_on:
            -  mysql
                
    mysql:
        image: mysql:5.7
        environment:
            MYSQL_ROOT_PASSWORD: ${DB_PASS}
            MYSQL_DATABASE: ${DB_NAME}
        volumes:
            - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```



