# INSTRUCTION.md


## Dockerfile for app:
# Stage 1: Build Stage
ARG PYTHON_VERSION=3.8
FROM python:${PYTHON_VERSION} as builder

# Set the working directory
WORKDIR /app
COPY . .

# Stage 2: Run Stage
FROM python:${PYTHON_VERSION} as run

WORKDIR /app

ENV PYTHONUNBUFFERED=1

COPY --from=builder /app .

RUN pip install --upgrade pip && \
    pip install -r requirements.txt

EXPOSE 8080

## Dockerfile for DB:
FROM mysql:latest

ENV MYSQL_ROOT_PASSWORD=1234
ENV MYSQL_DATABASE=app_db
ENV MYSQL_USER=app_user
ENV MYSQL_PASSWORD=1234

EXPOSE 3306

VOLUME /var/lib/mysql

ENTRYPOINT ["sh", "-c", "python manage.py migrate && python manage.py runserver 0.0.0.0:8080"]

## Compose file:
networks:
  todoapp_net:
    name: todoapp_net
    driver: bridge
services:
  todoapp_db:
    container_name: todoapp_db
    image: mysql:8.0
    ports:
      - "8000:8000"
      - "8001:3306"
    volumes:
      - todoapp_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=1234
      - MYSQL_DATABASE=app_db
      - MYSQL_USER=app_user
      - MYSQL_PASSWORD=1234
    networks:
      - todoapp_net
  todoapp:
    container_name: todoapp
    image: todoapp:2.0.1
    build:
      context: .
      args:
        - PYTHON_VERSION:3.9
    ports:
      - "8002:8080"
    networks:
      - todoapp_net
    depends_on:
      - todoapp_db
    restart: unless-stopped
volumes:
  todoapp_data:
    name: todoapp_data


## Running the application

1. Build and start containers in the background:

```
docker-compose up -d --build
```

2. Check that containers are running:

```
docker-compose ps
```

3. View logs if something is not working:

```
docker-compose logs -f
```

---

## Stopping the application

1. Stop containers:

```
docker-compose stop
```

2. Stop and remove containers, networks, and default resources:

```
docker-compose down
```

3. Stop everything **and remove volumes** (this deletes database data):

```
docker-compose down -v
```

---

## Useful commands

Rebuild a single service:

```
docker-compose build todoapp
```

Restart a service:

```
docker-compose restart todoapp
```

Open a shell inside a container:

```
docker exec -it todoapp sh
```
