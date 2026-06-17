# Multi-Stage Builds & Docker Hub

## Multi-Stage Build

**Frontend**

```
#stage 1

FROM node:20-alpine AS prod

WORKDIR /app

RUN addgroup -S app && adduser -S -G app app

COPY . .

RUN npm install

RUN npm run build

#stage 2


FROM nginx:stable-alpine
# Copy custom Nginx configuration file
COPY nginx.conf /etc/nginx/conf.d/default.conf
# Copy built static files from Stage 1
COPY --from=prod /app/dist /usr/share/nginx/html


EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```

**Backend**

```

#stage 1

FROM golang:1.22-alpine AS builder

WORKDIR /app

RUN addgroup -S app && adduser -S -G app app

COPY . .

RUN go mod download

RUN CGO_ENABLED=0 go build -o devboard


#stage 2 starting here
FROM alpine:3.20

WORKDIR /app

COPY --from=builder /app/devboard .

RUN addgroup -S app && adduser -S -G app app

RUN chown -R app:app /app

USER app

EXPOSE 8080

CMD [ "./devboard" ]

```

**Docker compose file**

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
    image: devboard-backend
    networks:
      - devboard-net
    environment:
      port: 8080
      POSTGRES_URL: postgres://devboard:devboard@postgres:5432/devboard?sslmode=disable
    ports:
      - "8080:8080"
    restart: on-failure
    depends_on:
      postgres:
        condition: service_healthy
    
  frontend:
    image: devboard-frontend
    networks:
      - devboard-net
    ports:
      - "80:80"
    restart: on-failure
    depends_on:
      - backend
    
networks:
    devboard-net:
      driver: bridge

volumes:
    pgdata:

```

## Task 3: Push to Docker Hub

```
   74  docker login
   75  docker images
   76  docker tag devboard-backend:latest rohit5126/devboard-backend:latest
   77  docker images
   78  docker push rohit5126/devboard-backend:latest
```
