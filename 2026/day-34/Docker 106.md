# Docker Compose: Real-World Multi-Container Apps

## Task 2: depends_on & Healthchecks

**depends-on**
```
services:
  web:
    image: nginx:latest
    depends_on:
      - db
      - redis

  db:
    image: postgres:15

  redis:
    image: redis:alpine
```
The basic syntax only ensures the dependency container has started (its process was triggered). It does not guarantee that the service inside it.

**The Solution: Advanced Health Check Conditions**

```
services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy  # Waits for the DB health check to succeed

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydatabase"]
      interval: 5s
      timeout: 5s
      retries: 5
```

this ensure the health of Postgres db is healthy and the only wb service will start.


## Task 3: Restart Policies

Docker restart policies control whether the Docker daemon automatically restarts containers after they exit or when the host system reboots

<img width="817" height="505" alt="image" src="https://github.com/user-attachments/assets/5850cfa6-91bd-4f68-acda-8412bb8d342b" />

**addgroup -S app && adduser -S -G app app**

This command creates a new system group named app and then creates a system user also named app who belongs to that group. It is a highly optimized pattern commonly used in Alpine Linux Dockerfiles to run applications securely without root privileges.

## Named Networks & Volumes
 **labels are key-value pairs used to add metadata to your containers, networks, and volumes**

```
services:
  postgres:
    image: postgres:16-alpine
    networks:
      - devboard-net
    environment:
      POSTGRES_USER: devboard
      POSTGRES_DB: devboard
      POSTGRES_PASSWORD: devboard
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init/postgres:/docker-entrypoint-initdb.d:ro
    restart: on-failure
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydatabase"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    networks:
      - devboard-net
    environment:
      port: 8080
      POSTGRES_URL: postgres://devboard:devboard@postgres:5432/devboard?sslmode=disable
    ports:
      - "8080:8080"
    restart: on-failure
    depends_on:
      - postgres
    
  frontend:
    build: ./frontend
    networks:
      - devboard-net
    ports:
      - 4173:4173
    restart: on-failure
    depends_on:
      - backend
    
networks:
    devboard-net:
      driver: bridge

volumes:
    pgdata:

```
