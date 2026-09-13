# Dockerfile

A **Dockerfile** is a text file that contains instructions used to build a Docker image.

Docker reads the Dockerfile **from top to bottom** and executes its instructions during the image build process.

---

## `FROM`

Defines the base image used to build the image.

```dockerfile
FROM <image>:<tag>
```

Example:

```dockerfile
FROM ubuntu:24.04
```

`FROM` is normally the first instruction in a Dockerfile.

---

## `ARG`

Defines a **build-time variable**.

```dockerfile
ARG NAME=VALUE
```

Example:

```dockerfile
ARG APP_VERSION=1.0
```

You can pass a value during the build:

```bash
docker build --build-arg APP_VERSION=2.0 -t myapp .
```

### Important

`ARG` is mainly available during the **image build process**.

---

## `WORKDIR`

Sets the working directory inside the image.

```dockerfile
WORKDIR /app
```

After setting `WORKDIR`, subsequent instructions use this directory as their working directory when applicable.

Example:

```dockerfile
FROM ubuntu:24.04

WORKDIR /app

COPY . .
```

---

## `COPY`

Copies files or directories from the **build context** into the image.

```dockerfile
COPY <source> <destination>
```

Example:

```dockerfile
COPY app.py /app/
```

If `WORKDIR` is already set:

```dockerfile
WORKDIR /app

COPY . .
```

This copies the contents of the build context into `/app`.

---

## `ADD`

Copies files or directories into the image.

```dockerfile
ADD <source> <destination>
```

Example:

```dockerfile
ADD app.py /app/
```

`ADD` provides additional functionality compared with `COPY`.

For ordinary file copying, `COPY` is generally preferred because its behavior is more explicit.

---

## `LABEL`

Adds metadata to the Docker image.

```dockerfile
LABEL key="value"
```

Example:

```dockerfile
LABEL version="1.0"
LABEL maintainer="developer"
```

Labels can be viewed using:

```bash
docker image inspect <image>
```

---

## `ENV`

Defines an environment variable in the image.

```dockerfile
ENV NAME=value
```

Example:

```dockerfile
ENV APP_ENV=production
```

The variable will be available in containers created from the image.

---

## `RUN`

Executes a command **during the image build process**.

```dockerfile
RUN <command>
```

Example:

```dockerfile
RUN apt update && apt install -y python3
```

`RUN` is executed when you run:

```bash
docker build
```

It is **not** executed every time the container starts.

Each `RUN` instruction can create an image layer.

---

## `SHELL`

Changes the default shell used by shell-form instructions.

```dockerfile
SHELL ["/bin/bash", "-c"]
```

Example:

```dockerfile
FROM ubuntu:24.04

SHELL ["/bin/bash", "-c"]

RUN echo "Hello"
```

This is useful when you need to use a shell other than the default shell.

---

## `EXPOSE`

Documents the port that the application inside the container is expected to listen on.

```dockerfile
EXPOSE <port>
```

Example:

```dockerfile
EXPOSE 80
```

### Important

`EXPOSE` **does not publish the port to the host**.

To publish the port, use `-p` when running the container:

```bash
docker run -p 8080:80 myimage
```

Meaning:

```text
Host port 8080 → Container port 80
```

---

## `USER`

Specifies the user used to run subsequent Dockerfile instructions and the container's default process.

```dockerfile
USER <username>
```

Example:

```dockerfile
USER appuser
```

It can be used to avoid running the application as `root`.

Example:

```dockerfile
FROM ubuntu:24.04

RUN useradd -m appuser

USER appuser

CMD ["bash"]
```

---

# `CMD`

Defines the **default command** for a container created from the image.

```dockerfile
CMD ["command", "argument"]
```

Example:

```dockerfile
CMD ["python3", "app.py"]
```

The `CMD` instruction can be overridden when running the container.

Example:

```bash
docker run myimage bash
```

If the image has:

```dockerfile
CMD ["python3", "app.py"]
```

the `bash` command replaces the default `CMD`.

---

# `ENTRYPOINT`

Defines the main executable of the container.

```dockerfile
ENTRYPOINT ["executable", "argument"]
```

Example:

```dockerfile
ENTRYPOINT ["python3"]
```

Then:

```bash
docker run myimage app.py
```

results in:

```text
python3 app.py
```

---

# `CMD` + `ENTRYPOINT`

They can be used together.

Example:

```dockerfile
ENTRYPOINT ["python3"]

CMD ["app.py"]
```

Running:

```bash
docker run myimage
```

results in:

```text
python3 app.py
```

While:

```bash
docker run myimage test.py
```

results in:

```text
python3 test.py
```

A useful mental model:

```text
ENTRYPOINT → main executable
CMD        → default arguments
```

---

# Build Arguments vs Environment Variables

### `ARG`

```dockerfile
ARG APP_VERSION=1.0
```

Primarily used during:

```text
docker build
```

### `ENV`

```dockerfile
ENV APP_ENV=production
```

Available in the resulting image/container environment.

Mental model:

```text
ARG → build-time configuration
ENV → runtime environment
```

---

# `CMD` vs `ENTRYPOINT`

### `CMD`

Provides a default command or default arguments.

```dockerfile
CMD ["python3", "app.py"]
```

It can be overridden by providing another command to `docker run`.

### `ENTRYPOINT`

Defines the executable that the container is intended to run.

```dockerfile
ENTRYPOINT ["python3"]
```

Together:

```dockerfile
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

```text
ENTRYPOINT + CMD
      ↓
python3 app.py
```

---

# Dockerfile Build

To build an image from a Dockerfile:

```bash
docker build -t <image_name>:<tag> .
```

Example:

```bash
docker build -t myapp:v1 .
```

Here:

```text
-t myapp:v1 → image name and tag
.           → build context
```

---

# Example Dockerfile

```dockerfile
FROM python:3.12

WORKDIR /app

ARG APP_VERSION=1.0

LABEL version=$APP_VERSION

ENV APP_ENV=production

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

USER 1000

ENTRYPOINT ["python"]

CMD ["app.py"]
```

Build:

```bash
docker build -t myapp:v1 .
```

Run:

```bash
docker run -p 8000:8000 myapp:v1
```

---

# Dockerfile Instructions — Quick Reference

| Instruction  | Purpose                                                     |
| ------------ | ----------------------------------------------------------- |
| `FROM`       | Defines the base image                                      |
| `ARG`        | Defines a build-time variable                               |
| `WORKDIR`    | Sets the working directory                                  |
| `COPY`       | Copies files/directories into the image                     |
| `ADD`        | Copies files with additional functionality                  |
| `LABEL`      | Adds image metadata                                         |
| `ENV`        | Defines environment variables                               |
| `RUN`        | Executes commands during image build                        |
| `SHELL`      | Changes the default shell                                   |
| `EXPOSE`     | Documents a container port                                  |
| `USER`       | Sets the user for subsequent instructions/container process |
| `CMD`        | Defines the default command/arguments                       |
| `ENTRYPOINT` | Defines the main executable                                 |
