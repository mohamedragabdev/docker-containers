# Docker Compose

## 1. What is Docker Compose?

Docker Compose is a tool for defining and running **multi-container Docker applications** using a YAML configuration file.

Instead of managing several containers manually with separate `docker run` commands, you describe the application's services in one file.

For example, an application might contain:

```text
Web App
   |
   +---- Database
   |
   +---- Redis
   |
   +---- Worker
```

Compose lets you define these services together.

---

## 2. The Problem Compose Solves

Imagine you have two containers:

```text
container A → Web application
container B → Database
```

Without Compose, you may have to manually:

```bash
docker network create app-network

docker run -d \
  --name database \
  --network app-network \
  ...

docker run -d \
  --name web \
  --network app-network \
  -p 8000:8000 \
  ...
```

You have to remember:

- container names
- images
- ports
- networks
- volumes
- environment variables
- startup options

Compose lets you describe this configuration once.

---

# 3. Compose File

A Compose application is normally described in:

```text
compose.yaml
```

Example:

```yaml
services:

  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: app
```

This describes two services:

```text
web
database
```

Compose can create and run both.

---

# 4. What Is a Service?

A **service** is a definition of a container/application component.

Example:

```yaml
services:

  web:
    image: nginx:latest

  database:
    image: mysql:8
```

There are two services:

```text
web       → Nginx
database  → MySQL
```

Do not think of `service` as necessarily meaning a separate physical server.

In Compose, a service is essentially a definition from which Compose creates containers.

---

# 5. `image`

`image` specifies the Docker image used by the service.

```yaml
services:
  web:
    image: nginx:latest
```

Another example:

```yaml
services:
  database:
    image: mysql:8
```

Compose will use the specified image to create the container.

---

# 6. `build`

Instead of using an existing image, you can tell Compose to build an image from a Dockerfile.

```yaml
services:
  web:
    build: .
```

If the directory contains:

```text
project/
├── Dockerfile
└── compose.yaml
```

then:

```yaml
build: .
```

means:

> Build the image using the Dockerfile in this directory.

Conceptually:

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
Container
```

---

# 7. Dockerfile vs Compose

These are related but have different responsibilities.

## Dockerfile

Answers:

> How should I build my image?

Example:

```dockerfile
FROM nginx:latest

COPY ./html /usr/share/nginx/html
```

## Compose

Answers:

> What containers/services should run together, and how should they be configured?

Example:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
```

So:

```text
Dockerfile → builds the image

Compose → runs/coordinates the application
```

They are not competitors.

---

# 8. `ports`

`ports` publishes a container port to the host.

```yaml
ports:
  - "8080:80"
```

This means:

```text
HOST                CONTAINER
8080   ───────────> 80
```

So you can access:

```text
localhost:8080
```

and traffic is forwarded to port `80` inside the container.

The syntax is:

```text
HOST_PORT:CONTAINER_PORT
```

---

# 9. `environment`

Environment variables can be defined with:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: example
  MYSQL_DATABASE: app
```

This is similar to:

```bash
docker run \
  -e MYSQL_ROOT_PASSWORD=example \
  -e MYSQL_DATABASE=app \
  ...
```

Environment variables are commonly used for configuration.

---

# 10. `volumes`

Compose can configure Docker mounts.

### Bind mount

```yaml
volumes:
  - ./src:/app/src
```

Meaning:

```text
project/src
     ↓
container:/app/src
```

This is useful when developing because changes made on the host are visible inside the container.

### Named volume

```yaml
volumes:
  - database-data:/var/lib/mysql
```

This uses a Docker-managed named volume.

For example:

```yaml
services:
  database:
    image: mysql:8
    volumes:
      - database-data:/var/lib/mysql

volumes:
  database-data:
```

The top-level declaration:

```yaml
volumes:
  database-data:
```

defines the named volume.

---

# 11. Why Named Volumes Matter

The container and the volume are separate resources.

Conceptually:

```text
Container
    |
    | mounts
    v
database-data
```

If the container is removed:

```text
Container → removed
Volume    → remains
```

The volume can then be mounted into a new container.

This is particularly useful for persistent application data such as databases.

---

# 12. `networks`

Compose can define Docker networks.

Example:

```yaml
services:

  web:
    image: nginx:latest
    networks:
      - app-network

  database:
    image: mysql:8
    networks:
      - app-network

networks:
  app-network:
```

Both services are connected to:

```text
app-network
```

Therefore they can communicate over the Docker network.

---

# 13. Service-to-Service Communication

One of the most useful Compose features is Docker's internal DNS.

Suppose:

```yaml
services:

  web:
    image: some-web-image

  database:
    image: mysql:8
```

The application in `web` can normally reach the database using:

```text
database
```

as the hostname.

For example:

```text
DB_HOST=database
```

You do **not** need to manually discover and hardcode the database container's IP address.

Conceptually:

```text
web container
     |
     | hostname: database
     v
Docker DNS
     |
     v
database container
```

---

# 14. Default Network

You do not always need to define `networks:` yourself.

Compose automatically creates a default network for the application.

For example:

```yaml
services:

  web:
    image: nginx:latest

  database:
    image: mysql:8
```

Both services are normally connected to the same Compose-created network.

Therefore the `web` service can communicate with the `database` service using:

```text
database
```

A custom network is only necessary when you want explicit network configuration or multiple network segments.

---

# 15. `depends_on`

Example:

```yaml
services:

  web:
    image: my-web-image
    depends_on:
      - database

  database:
    image: mysql:8
```

This expresses a dependency:

```text
web
 ↓
database
```

It can affect the order in which Compose starts services.

Important:

`depends_on` does not by itself guarantee that the database application is fully ready to accept connections.

A container being started is not necessarily the same thing as its application being ready.

For readiness checks, Compose can use healthchecks.

---

# 16. `healthcheck`

A healthcheck tells Docker how to determine whether a service is healthy.

Generic example:

```yaml
services:

  database:
    image: mysql:8
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 5
```

The exact healthcheck depends on the application.

Healthchecks become useful when another service needs to wait for an application to actually become ready.

---

# 17. `command`

You can override the command used to start a service.

Example:

```yaml
services:
  app:
    image: python:3.13
    command: python app.py
```

This is conceptually similar to providing a command after the image in:

```bash
docker run IMAGE COMMAND
```

You can also write:

```yaml
command:
  - python
  - app.py
```

---

# 18. `restart`

You can configure a restart policy:

```yaml
restart: unless-stopped
```

Common policies include:

```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

This controls what Docker should do when a container exits.

---

# 19. Basic Compose Commands

## Start

```bash
docker compose up
```

Compose creates and starts the required services.

---

## Start in background

```bash
docker compose up -d
```

`-d` means detached mode.

---

## Build and start

If a service uses `build`:

```bash
docker compose up --build
```

Or:

```bash
docker compose up -d --build
```

---

## Stop and remove the application

```bash
docker compose down
```

Normally this removes:

```text
Compose containers
Compose network
```

but does not remove named volumes by default.

---

## Remove volumes too

```bash
docker compose down -v
```

This also removes the Compose-managed named volumes.

Be careful with this when those volumes contain important data.

---

# 20. `docker compose ps`

Shows the services/containers belonging to the Compose project:

```bash
docker compose ps
```

Useful for checking whether services are running.

---

# 21. `docker compose logs`

Show logs:

```bash
docker compose logs
```

Show logs for one service:

```bash
docker compose logs web
```

Follow logs continuously:

```bash
docker compose logs -f web
```

---

# 22. `docker compose exec`

Execute a command inside an already-running service container.

Example:

```bash
docker compose exec web bash
```

If the service is called `app`:

```bash
docker compose exec app bash
```

You can also execute a command directly:

```bash
docker compose exec app python app.py
```

The important distinction:

```text
exec
→ execute inside an existing running container
```

---

# 23. `docker compose run`

`run` is for running a one-off command using a service definition.

Example:

```bash
docker compose run app python script.py
```

Conceptually:

```text
compose service definition
          ↓
temporary/one-off container
          ↓
command
```

Compare:

```bash
docker compose exec app ...
```

with:

```bash
docker compose run app ...
```

`exec` uses an existing running container.

`run` creates a container for the one-off command.

---

# 24. `docker compose build`

Build services that have a `build` configuration:

```bash
docker compose build
```

Specific service:

```bash
docker compose build web
```

Without cache:

```bash
docker compose build --no-cache
```

---

# 25. `docker compose pull`

Pull images used by the Compose file:

```bash
docker compose pull
```

Useful when services use images from a registry.

---

# 26. `docker compose config`

Validate and render the Compose configuration:

```bash
docker compose config
```

This is useful when debugging YAML or Compose configuration.

---

# 27. A General Example

Consider a generic application with:

```text
Frontend
Backend
Database
Redis
```

A Compose file could look like:

```yaml
services:

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      DATABASE_HOST: database
      REDIS_HOST: redis
    depends_on:
      - database
      - redis

  database:
    image: postgres:17
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: example
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:latest

volumes:
  postgres-data:
```

The important part is understanding the structure, not memorizing this exact file.

Conceptually:

```text
                  Compose
                     |
       +-------------+-------------+
       |             |             |
   frontend       backend       database
                     |
                     |
                   redis

database
    |
    v
postgres-data
```

---

# 28. What Happens When You Run `docker compose up`?

Given:

```yaml
services:

  web:
    image: nginx:latest

  database:
    image: mysql:8
```

and:

```bash
docker compose up -d
```

Compose determines what resources are required.

Conceptually:

```text
compose.yaml
     |
     +---- web service
     |
     +---- database service
     |
     +---- network
     |
     +---- volumes, if defined
```

Then it creates/starts the required containers and connects them according to the configuration.

---

# 29. Compose Project

Compose treats the collection of services as an application/project.

For example:

```text
my-project/
└── compose.yaml
```

Running:

```bash
docker compose up -d
```

manages the services defined in that Compose file as a group.

This is useful because you can start or stop the application's infrastructure together.

---

# 30. Compose and Container Names

You can explicitly specify:

```yaml
container_name: my-container
```

Example:

```yaml
services:
  web:
    image: nginx
    container_name: my-web
```

However, `container_name` is optional.

Compose can generate container names automatically.

For normal Compose projects, there is often little reason to hardcode container names.

---

# 31. A Useful Mental Model

Think about Compose as a description of the **runtime architecture** of an application.

```text
                  compose.yaml
                       |
        +--------------+--------------+
        |              |              |
      service        service        service
        |              |              |
     container      container      container
        |
      ports
        |
     volumes
        |
     networks
        |
   environment
```

Dockerfile answers:

```text
How do I build this image?
```

Compose answers:

```text
How do these application components run together?
```

---

# 32. Compose vs Docker Run

With `docker run`, you manually specify container configuration:

```bash
docker run \
  --name web \
  --network app-network \
  -p 8080:80 \
  nginx
```

With Compose:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    networks:
      - app-network
```

The Compose file becomes a reusable description of the setup.

---

# 33. Compose vs Docker Swarm

They are different tools/concepts.

### Compose

Focuses on defining and running a multi-container application.

```text
Application
   |
   +-- Web
   +-- Database
   +-- Redis
```

### Swarm

Is an orchestration system for running services across a cluster of Docker nodes.

```text
Node 1 ─┐
Node 2 ─┼── Swarm cluster
Node 3 ─┘
```

Compose is primarily about the application's container setup.

Swarm adds cluster orchestration capabilities.

---

# 34. Compose vs Kubernetes

Compose is much simpler than Kubernetes.

Compose is commonly useful for:

- local development
- testing
- small multi-container environments
- quickly reproducing an application's infrastructure

Kubernetes is a much larger orchestration platform with concepts such as:

- clusters
- nodes
- pods
- deployments
- services
- scheduling
- scaling
- self-healing
- rolling updates

Do not think of Compose as a smaller syntax for Kubernetes. They solve related but different levels of problems.

---

# 35. Important Things to Remember

### 1. `services`

Defines the application's components.

```yaml
services:
  web:
  database:
```

### 2. `image`

Uses an existing image.

```yaml
image: nginx:latest
```

### 3. `build`

Builds an image using a Dockerfile.

```yaml
build: .
```

### 4. `ports`

Publishes container ports to the host.

```yaml
ports:
  - "8080:80"
```

### 5. `environment`

Provides environment variables.

```yaml
environment:
  APP_ENV: development
```

### 6. `volumes`

Configures bind mounts or named volumes.

```yaml
volumes:
  - data:/data
```

### 7. `networks`

Controls which services communicate through a Docker network.

```yaml
networks:
  - app-network
```

### 8. `depends_on`

Expresses service startup dependencies.

```yaml
depends_on:
  - database
```

### 9. `command`

Overrides the container's command.

```yaml
command: python app.py
```

---

# 36. Core Command Cheat Sheet

```bash
# Start
docker compose up

# Start in background
docker compose up -d

# Build and start
docker compose up -d --build

# Show running services
docker compose ps

# Show logs
docker compose logs

# Follow logs
docker compose logs -f

# Stop/remove Compose resources
docker compose down

# Stop/remove resources and named volumes
docker compose down -v

# Execute command in running service
docker compose exec SERVICE COMMAND

# Run one-off command
docker compose run SERVICE COMMAND

# Build images
docker compose build

# Pull images
docker compose pull

# Validate/render configuration
docker compose config
```

---

# 37. The Main Idea

You do not need to memorize every Compose option.

Understand this relationship:

```text
Dockerfile
   ↓
Build image

Compose
   ↓
Define services
   ↓
Configure containers
   ↓
Configure ports
   ↓
Configure volumes
   ↓
Configure networks
   ↓
Configure environment
   ↓
Run the application as a group
```

The key distinction is:

```text
Docker
→ manages containers/images/volumes/networks

Dockerfile
→ defines how an image is built

Docker Compose
→ defines how multiple application services are run together
```

Once this mental model is clear, the individual Compose options become much easier to understand.
