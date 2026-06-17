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


# Task 3: Restart Policies

Docker restart policies control whether the Docker daemon automatically restarts containers after they exit or when the host system reboots

<img width="817" height="505" alt="image" src="https://github.com/user-attachments/assets/5850cfa6-91bd-4f68-acda-8412bb8d342b" />





