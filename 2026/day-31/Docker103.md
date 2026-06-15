## Task 1: Your First Dockerfile

**Dockerfile**
```
FROM ubuntu 

WORKDIR /app

RUN apt-get update

RUN apt-get install curl -y

CMD [ "echo", "Hello from my custom image!" ]
```
```bash
 2035  vim Dockerfile 
 2036  docker build -t ubuntu:v1 .
 2037  docker run -d ubuntu:v1 
 2038  docker logs unruffled_proskuriakova 

 ```

 ## Task 2: Dockerfile Instructions

 ```
 FROM python:3.9

WORKDIR /app

COPY requirements.txt /app

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*    

RUN pip install mysqlclient

RUN pip install --no-cache-dir -r requirements.txt

COPY . /app

EXPOSE 8000

CMD python /app/manage.py runserver 0.0.0.0:8000
```

## Task 3: CMD vs ENTRYPOINT

<img width="810" height="472" alt="image" src="https://github.com/user-attachments/assets/64b3a263-6327-4c2b-aa3c-2f8e587d4d7a" />

## Build a Simple Web App Image

```
FROM nginx:latest 

WORKDIR /app

RUN apt-get update

COPY ./index.html /usr/share/nginx/html

EXPOSE 80
```

## Task 5: .dockerignore

**Layer order dictates build speed in Docker because Docker caches each step. If a layer's contents change, Docker invalidates that layer and all subsequent layers.
To maximize speed, place least-frequently changing instructions first and frequently changing instructions last**
