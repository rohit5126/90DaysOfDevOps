# Docker Revision & Cheat Sheet
Self-Assessment Checklist
Mark yourself honestly — can do, shaky, or haven't done:

## Run a container from Docker Hub (interactive + detached)
 ```
docker run -d -it ubuntu
```

 ## List, stop, remove containers and images
 ```
docker ps
docker stop
docker rm
docker rmi
docker images
docker image prune -y
docker conatiner prune -y
```


## Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD
 ```
FROM nginx:latest

WORKDIR /app

COPY nginx.conf /var/lib/nginx/

COPY index.html /var/www/html/

RUN systemd nginx restart

CMD [ "nginx" ,"-g", "daemon off" ]
```


##  Build and tag a custom image

 docker build -t my-image .

 ## Create and use named volumes
 ```
docker volume create pg-data
```

 ## Use bind mounts

 ```
docker run -d -p 80:80 -v /home/ubuntu/archive:/app/data my-image.
```

 ## Create custom networks and connect containers
```
 docker netwrok create my-net

docker run -d -p 80:80 --network my-net my-image.
```

## Write a docker-compose.yml for a multi-container app & Use healthchecks and depends_on
```
service:
  db-app:
    image: postgres
    environments:
      POSTGRES_USER: ${POST_USER}
      POSTGRES_PASSWORD: ${POST_PASS}
      POSTGRES_DB: ${POST_DB}
    volumes:
      - pg-data:var/lib/postgresql/data
    network:
      - my-net
    ports: 
      - "5432:5432"
    restart: on-failure
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydatabase" ]
      interval: 5s
      timeout: 10s
      retries: 5

  backend:
    build: ./backend
    environment:
      ports: 8080
      POSTGRES_URL: postgres://database:pssword@db-app:5432/mydbapp?sslmode=disable
    networks:
      - my-net
    ports: 
      - "8080:8080"
    depends_on:
      db_app:
        condition: service_healthy

  frontend:
    build: ./frontend
    ports: 
      - "4173:4173"
    networks:
      - my-net
    depends_on:
      - backend

volumes:
  pg-data:

networks:
  my-net:
    driver: bridge

```



## Use environment variables and .env files in Compose
.env
```
POST_USER: database
POST_PASS: passowrd
POST_DB: mydbapp
```

 ## Write a multi-stage Dockerfile
```
FROM node-16:alpine AS prod

WORKDIR /app

COPY . .

RUN npm install && npm run build

FROM node-20:alpine

WORKDIR /app

COPY --from=prod /app/dist ./dist

COPY package*.json ./

RUN npm install -g vite

EXPOSE 4173

CMD ["npm","preview","--","--host","0.0.0.0","--port","4173"]

```

## Push an image to Docker Hub

```
docker login
make sure image name is docker_username/image_name:latest
docker push <image>
 ```

Quick-Fire Questions
Answer from memory, then verify:

What is the difference between an image and a container?
What happens to data inside a container when you remove it?
How do two containers on the same custom network communicate?
What does docker compose down -v do differently from docker compose down?
Why are multi-stage builds useful?
What is the difference between COPY and ADD?
What does -p 8080:80 mean?
How do you check how much disk space Docker is using?


