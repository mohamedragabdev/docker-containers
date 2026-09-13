# Docker Notes

# 1. Docker Groups

## `getent group docker`

Shows information about the `docker` Linux group.

```bash
getent group docker
```

---

## `groups`

Shows the groups that the current user belongs to.

```bash
groups
```

---

## Add a User to the Docker Group

```bash
sudo usermod -aG docker <username>
```

### Options

```text
-a → append the group without removing existing supplementary groups
-G → specify supplementary groups
docker → group to add the user to
<username> → target user
```

After adding the user to the Docker group, log out and log back in for the group membership to take effect.

---

# 2. Docker Information

## `docker info`

Displays detailed information about the Docker installation and Docker daemon.

```bash
docker info
```

It can show information about:

* Docker Server
* Containers
* Images
* Storage Driver
* Logging Driver
* Docker Root Directory
* Plugins
* Security options
* CPU / Memory information
* Docker Swarm status

---

# 3. Docker Images

Commands beginning with:

```bash
docker image
```

are used to manage Docker images.

A **Docker image** is a read-only template used to create containers.

---

## Pull an Image

```bash
docker image pull [accountname/]image_name[:tag]
```

Examples:

```bash
docker image pull ubuntu
docker image pull ubuntu:24.04
docker image pull nginx:latest
docker image pull username/my-image:v1
```

### Image name structure

```text
[registry/] [namespace/] image[:tag]
```

For example:

```text
docker.io/username/my-image:v1
         ↓        ↓        ↓
      registry  image    tag
               name
```

In the simpler form:

```text
username/my-image:v1
```

* `username/` → Docker Hub namespace/account
* `my-image` → image name
* `:v1` → tag/version

If no tag is specified, Docker normally uses:

```text
latest
```

---

## List Images

```bash
docker image ls
```

Shows Docker images available locally.

---

# 4. Docker Containers

Commands beginning with:

```bash
docker container
```

are used to manage Docker containers.

A **container** is an instance created from a Docker image.

A container can be:

```text
Created
Running
Stopped
Exited
```

---

# 5. Create a Container

## `docker container create`

Creates a container **without starting it**.

```bash
docker container create -it <image_name> <command>
```

Example:

```bash
docker container create -it ubuntu bash
```

### `-i`

Keeps STDIN open.

```text
-i → interactive
```

### `-t`

Allocates a pseudo-terminal.

```text
-t → TTY
```

Together:

```text
-it → interactive terminal
```

The `create` command:

```text
Image
  ↓
Container created
  ↓
Container NOT started
```

---

# 6. List Containers

## All Containers

```bash
docker container ls -a
```

Shows all containers, including:

* Running containers
* Stopped containers
* Exited containers
* Created containers

---

## Running Containers Only

```bash
docker container ls
```

Shows only currently running containers.

---

# 7. Start an Existing Container

## `docker container start`

Starts a container that already exists.

```bash
docker container start <container_name_or_id>
```

For interactive use:

```bash
docker container start -i <container_name_or_id>
```

Example:

```bash
docker container start -i my-ubuntu
```

---

## `docker start -ai`

Shortcut form:

```bash
docker start -ai <container_name>
```

Equivalent idea:

```text
-a → attach to the container
-i → keep STDIN open
```

Example:

```bash
docker start -ai my-ubuntu
```

Used to start an existing container and attach your terminal to it.

---

# 8. Create + Start a Container

## `docker container run`

`docker run` creates **and starts** a new container.

Conceptually:

```text
docker run
    ↓
pull image if necessary
    ↓
create container
    ↓
start container
    ↓
run main process
```

Example:

```bash
docker container run -it ubuntu
```

or:

```bash
docker run -it ubuntu
```

---

# 9. Container Name

Use `--name` to assign a custom name.

```bash
docker run --name <container_name> -it <image>
```

Example:

```bash
docker run --name my-ubuntu -it ubuntu
```

Without `--name`, Docker automatically generates a name.

---

# 10. Run a Specific Command

General structure:

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Example:

```bash
docker run --name my-ubuntu -it ubuntu bash
```

Here:

```text
ubuntu → image
bash   → command executed inside the container
```

The command becomes the main process of the container.

---

# 11. `-it`

Used when you want an interactive terminal.

```bash
docker run -it ubuntu bash
```

Equivalent to:

```text
-i → interactive / keep STDIN open
-t → allocate a pseudo-TTY
```

---

# 12. `-d`

Runs the container in **detached mode**, meaning in the background.

```bash
docker run -d nginx
```

The terminal is returned to you while the container continues running in the background.

You can check it with:

```bash
docker ps
```

---

# 13. `--restart`

Defines the container's restart policy.

```bash
docker run --restart <policy> <image>
```

Example:

```bash
docker run --restart unless-stopped nginx
```

Common policies:

```text
no
always
on-failure
unless-stopped
```

### `no`

Do not automatically restart the container.

### `always`

Always restart the container when Docker restarts or when the container exits.

### `on-failure`

Restart only when the container exits with a non-zero exit code.

It can also accept a maximum retry count:

```bash
--restart on-failure:5
```

### `unless-stopped`

Restart the container unless it was explicitly stopped.

---

# 14. `-p` — Publish Ports

Maps a host port to a container port.

```bash
docker run -p <host_port>:<container_port> <image>
```

Example:

```bash
docker run -p 8080:80 nginx
```

Meaning:

```text
Host
Port 8080
   ↓
Container
Port 80
```

So requests sent to:

```text
localhost:8080
```

can reach port `80` inside the container.

---

# 15. Container ≠ Command

A container is **not the command itself**.

The container is the environment/process instance created from an image.

The container has a **main process**.

Example:

```bash
docker run -it ubuntu bash
```

Here:

```text
Image      → ubuntu
Container  → created from ubuntu
Command    → bash
Main       → bash process
```

### Important

If the container's main process exits, the container stops.

Example:

```bash
docker run ubuntu
```

The default process eventually exits, so the container stops.

However:

```text
Stopped container ≠ deleted container
```

The container still exists.

You can see it with:

```bash
docker container ls -a
```

And start it again:

```bash
docker container start <container_name_or_id>
```

---

# 16. Execute a Command Inside a Running Container

## `docker exec`

Runs a new process inside an already-running container.

```bash
docker exec <container_name_or_id> <command>
```

Example:

```bash
docker exec my-container ls
```

For an interactive shell:

```bash
docker exec -it my-container bash
```

Important distinction:

```text
docker run
→ creates a new container

docker exec
→ runs a new process inside an existing running container
```

---

# 17. Environment Variables with `-e`

The original notes contain:

```text
docker container {-e}
```

The `-e` option is used to define an environment variable inside the container.

Syntax:

```bash
docker run -e VARIABLE=value <image>
```

Example:

```bash
docker run -e APP_ENV=production myimage
```

Inside the container:

```bash
echo $APP_ENV
```

Output:

```text
production
```

You can define multiple environment variables:

```bash
docker run \
  -e APP_ENV=production \
  -e APP_PORT=8000 \
  myimage
```

---

# 18. Copy Files Between Host and Container

## `docker container cp`

Copies files or directories between the host and a container.

### Host → Container

```bash
docker container cp <source> <container_name>:<destination>
```

Example:

```bash
docker container cp app.py my-container:/app/
```

### Container → Host

```bash
docker container cp <container_name>:<source> <destination>
```

Example:

```bash
docker container cp my-container:/app/output.txt .
```

Mental model:

```text
Host → Container
Container → Host
```

---

# 19. `ping`

`ping` is a Linux networking command used to test network reachability.

Syntax:

```bash
ping <hostname_or_ip>
```

Example:

```bash
ping 8.8.8.8
```

or:

```bash
ping google.com
```

The original note:

```text
ping <pingnum>
```

is better understood as:

```bash
ping <hostname_or_ip>
```

You can limit the number of packets with:

```bash
ping -c <number> <hostname_or_ip>
```

Example:

```bash
ping -c 4 google.com
```

---

# 20. `--add-host`

Adds a custom hostname-to-IP mapping to the container's `/etc/hosts`.

Syntax:

```bash
docker run --add-host <hostname>:<ip_address> <image>
```

Example:

```bash
docker run \
  --add-host api.example.com:192.168.1.10 \
  nginx
```

Inside the container:

```text
api.example.com → 192.168.1.10
```

This is useful when you need a container to resolve a specific hostname to a specific IP address.

---

# 21. `--network`

Connects the container to a specific Docker network.

```bash
docker run --network <network_name> <image>
```

Example:

```bash
docker run \
  --network my-network \
  --name app \
  myimage
```

This allows the container to participate in the specified Docker network.

---

# 22. `-v` — Volume / Bind Mount

Mounts storage from the host into the container.

Syntax:

```bash
docker run -v <host_path>:<container_path> <image>
```

Example:

```bash
docker run \
  -v /home/user/data:/app/data \
  myimage
```

Meaning:

```text
Host:
/home/user/data

        ↓ mounted into

Container:
/app/data
```

The container can access the mounted files through:

```text
/app/data
```

The original note:

```text
-v path:dir_path_in_container
```

refers to this host-path → container-path mapping.

---

# 23. Combining `docker run` Options

A container can use several options at the same time.

Example:

```bash
docker container run \
  -d \
  --name my-app \
  --add-host api.example.com:192.168.1.10 \
  --network my-network \
  myimage
```

With a volume:

```bash
docker container run \
  -d \
  -v /host/path:/container/path \
  --name my-app \
  --add-host api.example.com:192.168.1.10 \
  --network my-network \
  myimage
```

---

# 24. Build an Image

## `docker build`

Builds a Docker image from a Dockerfile.

```bash
docker build -t <image_name>:<tag> .
```

Example:

```bash
docker build -t myapp:v1 .
```

Meaning:

```text
-t myapp:v1
→ image name + tag

.
→ build context
```

---

# 25. Docker Networks

Docker networks allow containers to communicate with each other.

## List Networks

```bash
docker network ls
```

Shows the Docker networks currently available.

---

## Create a Network

```bash
docker network create <network_name>
```

Example:

```bash
docker network create my-network
```

---

## Create a Network with a Subnet

```bash
docker network create --subnet <subnet> <network_name>
```

Example:

```bash
docker network create --subnet 172.20.0.0/16 my-network
```

The subnet defines the IP address range used by the network.

---

## `--internal`

Creates an internal network that is isolated from external networks.

Example:

```bash
docker network create --internal my-network
```

An internal network is useful when containers should communicate with each other without having direct external network connectivity through that network.

---

## Connect a Container to a Network

```bash
docker network connect <network_name> <container_name>
```

Example:

```bash
docker network connect my-network my-container
```

A running container can be connected to an additional network.

---

## Disconnect a Container from a Network

```bash
docker network disconnect <network_name> <container_name>
```

Example:

```bash
docker network disconnect my-network my-container
```

---

# 26. Docker Volumes

## Create a Volume

The original note:

```text
docker voluom create <voloum_name>
```

Correct command:

```bash
docker volume create <volume_name>
```

Example:

```bash
docker volume create my-volume
```

A Docker volume is Docker-managed persistent storage.

Unlike the container's writable layer, a volume can persist independently of the container.

---

# 27. Commit a Container to an Image

## `docker commit`

Creates a new image from the current state of a container.

Syntax:

```bash
docker commit <container_name> <new_image_name>
```

Example:

```bash
docker commit my-container myimage:v1
```

Conceptually:

```text
Running/stopped container
          ↓
    docker commit
          ↓
      New image
```

`docker commit` is useful for capturing a container's filesystem state, but for reproducible builds, a Dockerfile is generally preferred.

---

# 28. Docker Registry / Docker Hub

## `docker login`

Authenticates Docker CLI with a container registry.

```bash
docker login
```

After authentication, you can push images to a registry for which you have permission.

---

## `docker tag`

Creates another tag/reference for an image.

Syntax:

```bash
docker tag <source_image> <new_image>
```

Example:

```bash
docker tag myapp:v1 username/myapp:v1
```

Now the image can be referenced as:

```text
username/myapp:v1
```

This is commonly used before pushing an image to Docker Hub.

---

## `docker image push`

Pushes an image to a container registry.

```bash
docker image push <image_name>:<tag>
```

Example:

```bash
docker image push username/myapp:v1
```

Typical workflow:

```text
docker login
      ↓
docker build
      ↓
docker tag
      ↓
docker push
```

---

# 29. Docker Compose

Docker Compose is used to define and manage **multi-container applications**.

## Start Services

```bash
docker compose up
```

Modern Docker uses:

```bash
docker compose
```

rather than the older:

```bash
docker-compose
```

Detached mode:

```bash
docker compose up -d
```

---

## Stop and Remove Compose Resources

```bash
docker compose down
```

Stops and removes the containers and networks created by the Compose project.

---

## Remove Images with `down`

The original note:

```text
docker-compose down rmi
```

Correct syntax:

```bash
docker compose down --rmi all
```

This removes the images used by the Compose project as well.

---

## List Compose Services

```bash
docker compose ps
```

Shows the containers/services belonging to the Compose project.

---

# 30. Docker Swarm

Docker Swarm is Docker's native **container orchestration** system.

A Swarm consists of nodes that work together as a cluster.

There are two main node roles:

```text
Manager Node
Worker Node
```

---

# 31. Initialize Docker Swarm

The original note:

```text
docker swarm init --advertise-adder ip address:lestinPort
```

Correct form:

```bash
docker swarm init --advertise-addr <ip_address>:<port>
```

Example:

```bash
docker swarm init --advertise-addr 192.168.1.10:2377
```

### `--advertise-addr`

Specifies the IP address that the manager advertises to other Swarm nodes.

---

## `--listen-addr`

Specifies the address and port where the Swarm manager listens for cluster management traffic.

Syntax:

```bash
docker swarm init \
  --advertise-addr <ip_address>:<port> \
  --listen-addr <ip_address>:<port>
```

Example:

```bash
docker swarm init \
  --advertise-addr 192.168.1.10:2377 \
  --listen-addr 192.168.1.10:2377
```

---

# 32. Swarm Nodes

## `docker node ls`

Lists the nodes in the Swarm.

```bash
docker node ls
```

This command is normally executed on a **manager node**.

---

# 33. Swarm Services

A **service** defines the desired state for a workload running in Swarm.

## Create a Service

Original concept:

```text
docker service create <serviceName> -p<port> --replicas<number replicas> imageName
```

Correct syntax:

```bash
docker service create \
  --name <service_name> \
  -p <published_port>:<target_port> \
  --replicas <number> \
  <image>
```

Example:

```bash
docker service create \
  --name web \
  -p 8080:80 \
  --replicas 3 \
  nginx
```

This asks Swarm to maintain:

```text
3 replicas
```

of the service.

---

## List Services

```bash
docker service ls
```

Shows services running in the Swarm.

---

## Remove a Service

```bash
docker service rm <service_name>
```

Example:

```bash
docker service rm web
```

---

# 34. Scale a Swarm Service

The original note:

```text
docker service scale <serviceName> = number_of_scales
```

Correct syntax:

```bash
docker service scale <service_name>=<number_of_replicas>
```

Example:

```bash
docker service scale web=5
```

This changes the desired number of replicas to:

```text
5
```

---

# 35. Update a Swarm Service

The original note:

```text
docker service update --image <newImageName>
--update-parallelism <numberParallel>
--update-dalay<number-dalay>
<serviceName>
```

Correct syntax:

```bash
docker service update \
  --image <new_image> \
  --update-parallelism <number> \
  --update-delay <duration> \
  <service_name>
```

Example:

```bash
docker service update \
  --image myapp:v2 \
  --update-parallelism 2 \
  --update-delay 10s \
  web
```

### `--update-parallelism`

Controls how many tasks are updated at the same time.

### `--update-delay`

Controls the delay between update batches.

---

# 36. Roll Back a Swarm Service

```bash
docker service rollback <service_name>
```

Rolls the service back to its previous service configuration.

Example:

```bash
docker service rollback web
```

---

# 37. Stateless vs Stateful

The original notes:

```text
container is stateless
service usually is stateful
```

This needs an important distinction.

### Container

A container's writable filesystem is generally **ephemeral**.

If the container is removed, data stored only in its writable layer is lost.

Therefore, containers are commonly designed to be **stateless**.

Persistent data should normally be stored externally, for example in:

```text
Volumes
Databases
Object storage
External storage systems
```

### Service

A Docker Swarm **service is not inherently stateful**.

A service can run a stateless application or a stateful workload.

However, the original note likely refers to the idea that a service maintains a **desired state**.

For example:

```text
Desired state:
3 replicas
```

Swarm continuously attempts to keep:

```text
Running replicas = 3
```

---

# 38. Stateful Conditions

A workload is **stateful** when it needs to preserve data or state across restarts/replacements.

Examples:

```text
Database
File storage
Persistent application data
```

A stateless workload can generally be replaced without needing to preserve local container state.

---

# 39. Desired State

**Desired state** is the state that the orchestrator is instructed to maintain.

Example:

```text
Desired replicas = 3
```

If one replica fails:

```text
Desired state → 3
Current state → 2
```

Swarm attempts to create another task so that:

```text
Current state → 3
```

This desired-state model is fundamental to container orchestration.

---

# 40. Service Availability

The original note:

```text
service must have at laste one up on running all time
```

The idea is that a Swarm service has a desired number of replicas, and Swarm continuously attempts to maintain that desired state.

For example:

```bash
docker service create --name web --replicas 3 nginx
```

Desired state:

```text
3 running replicas
```

If one fails, Swarm attempts to replace it.

---

# 41. Docker Stack

```bash
docker stack
```

Docker Stack is used to deploy a **multi-service application to a Docker Swarm** using a Compose-style YAML file.

Common commands include:

```bash
docker stack deploy
docker stack ls
docker stack services
docker stack ps
docker stack rm
```

Example:

```bash
docker stack deploy -c docker-compose.yml my-stack
```

---

# 42. Docker Secrets

Docker Secrets provide a mechanism for storing sensitive data used by Swarm services.

Example:

```bash
docker secret create <secret_name> <file>
```

Example:

```bash
docker secret create db_password ./db_password.txt
```

Secrets can be used for sensitive information such as:

```text
Passwords
API keys
Certificates
Private credentials
```

The purpose is to avoid putting sensitive values directly into the image or normal configuration.

---

# 43. Kubernetes

The original notes then move from Docker Swarm into **Kubernetes**.

Kubernetes is a container orchestration platform.

It manages workloads using Kubernetes objects such as:

```text
Pods
Deployments
Services
Ingress
```

---

# 44. etcd

`etcd` is a distributed key-value store used by Kubernetes to store cluster state.

Conceptually:

```text
Kubernetes API Server
        ↓
      etcd
        ↓
Cluster state
```

It stores important Kubernetes configuration and state information.

---

# 45. Kubernetes Manager / Control Plane

The original note:

```text
manager node and structure
```

In Kubernetes terminology, the modern term is **Control Plane** rather than Swarm's "manager node".

The control plane contains components responsible for managing the cluster.

Important components include:

```text
kube-apiserver
etcd
kube-scheduler
kube-controller-manager
```

---

# 46. Kubernetes Worker Node

A worker node runs application workloads.

Important components include:

```text
kubelet
kube-proxy
Container runtime
```

### `kubelet`

The kubelet communicates with the Kubernetes control plane and ensures that the Pods assigned to the node are running as expected.

### `kube-proxy`

Provides networking functionality for Kubernetes Services and helps implement network traffic rules on nodes.

### Container Runtime

Responsible for running containers.

---

# 47. Kubernetes Uses a Container Runtime

The original note:

```text
kubernetes use runtime container
```

Kubernetes does not directly run containers itself.

It communicates with a **container runtime** through the Container Runtime Interface (CRI).

Examples of container runtimes include:

```text
containerd
CRI-O
```

---

# 48. Minikube

Minikube is used to run a Kubernetes cluster locally, mainly for learning and development.

It can run Kubernetes using a VM or a container-based driver, depending on the environment.

---

# 49. Minikube Start

The original note:

```text
minikube start --vm-driver | --drive=docker --cpus {number}
--memory {number}
```

The syntax should be understood as options rather than alternatives written with `|`.

Example:

```bash
minikube start --driver=docker --cpus=2 --memory=4096
```

### `--driver=docker`

Uses Docker as the Minikube driver.

### `--cpus`

Specifies the number of CPUs allocated to Minikube.

Example:

```bash
--cpus=2
```

### `--memory`

Specifies the amount of memory allocated.

Example:

```bash
--memory=4096
```

---

# 50. Minikube Status

```bash
minikube status
```

Shows the current status of the Minikube cluster.

---

# 51. Kubernetes Cluster Information

```bash
kubectl cluster-info
```

Displays information about the Kubernetes cluster and its endpoints.

---

# 52. Minikube Dashboard

```bash
minikube dashboard
```

Opens the Kubernetes Dashboard for the Minikube cluster when the dashboard addon is available/enabled.

---

# 53. `kubectl get`

Used to retrieve Kubernetes resources.

General form:

```bash
kubectl get <resource>
```

Examples:

```bash
kubectl get nodes
kubectl get pods
kubectl get all
```

---

## `-o wide`

Displays additional information.

Example:

```bash
kubectl get pods -o wide
```

For nodes:

```bash
kubectl get nodes -o wide
```

---

# 54. Kubernetes Layers / Objects

The original notes list:

```text
pods
deployment
service
ingress
```

These are important Kubernetes resources.

A simplified relationship:

```text
Deployment
    ↓
ReplicaSets
    ↓
Pods
    ↓
Containers
```

A Service provides stable network access to Pods.

Ingress provides HTTP/HTTPS routing into Services.

---

# 55. Pods

A **Pod** is the smallest deployable unit in Kubernetes.

A Pod can contain one or more containers.

Commonly:

```text
Pod
 └── Container
```

But a Pod can also contain multiple tightly coupled containers.

---

# 56. Deployment

A **Deployment** manages replicated Pods.

It helps provide:

* Desired number of replicas
* Rolling updates
* Rollbacks
* Replacement of failed Pods

---

# 57. Service

A Kubernetes **Service** provides a stable network endpoint for a set of Pods.

Pods are ephemeral and their IP addresses can change.

The Service provides a stable way to reach them.

---

# 58. Ingress

An **Ingress** defines HTTP/HTTPS routing rules for traffic entering the Kubernetes cluster.

Conceptually:

```text
Client
  ↓
Ingress
  ↓
Service
  ↓
Pods
```

---

# 59. `minikube ip`

```bash
minikube ip
```

Returns the IP address associated with the Minikube node.

This can be useful when accessing services exposed through the Minikube environment.

---

# 60. Create a Kubernetes Deployment

```bash
kubectl create deployment <deployment_name> --image=<image>
```

Example:

```bash
kubectl create deployment my-app --image=nginx
```

This creates a Deployment using the specified image.

---

# 61. `curl`

`curl` is a command-line tool used to make network requests.

Example:

```bash
curl http://example.com
```

The original note:

```text
curl <command><podname>
```

is incomplete.

A common use in Kubernetes is to send an HTTP request to a Pod or Service endpoint:

```bash
curl http://<address>:<port>
```

---

# 62. `kubectl proxy`

```bash
kubectl proxy
```

Starts a local proxy between your machine and the Kubernetes API server.

This can allow local access to Kubernetes API resources through the proxy.

---

# 63. Execute Commands Inside a Kubernetes Pod

The original note:

```text
kubectl exec -it|-d <podname> -- command bash
```

Correct general form:

```bash
kubectl exec -it <pod_name> -- <command>
```

Example:

```bash
kubectl exec -it my-pod -- bash
```

This opens an interactive shell if `bash` exists in the container.

### `-i`

Keep STDIN open.

### `-t`

Allocate a pseudo-terminal.

For a command without an interactive terminal:

```bash
kubectl exec <pod_name> -- ls
```

The original `-d` should not be treated as the Docker `-d` detached-mode equivalent for `kubectl exec`; `kubectl exec` has its own supported options and behavior.

---

# 64. Expose a Deployment as a Service

The original note:

```text
kubectl expose <deploymentname> --type="type name" --port <portnum>
```

General syntax:

```bash
kubectl expose deployment <deployment_name> \
  --type=<service_type> \
  --port=<port>
```

Example:

```bash
kubectl expose deployment my-app \
  --type=NodePort \
  --port=80
```

This creates a Kubernetes Service for the Deployment.

Common Service types include:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

# 65. Describe a Kubernetes Resource

```bash
kubectl describe <resource> <name>
```

Example:

```bash
kubectl describe service my-service
```

This displays detailed information about the resource.

The original note:

```text
kubectl describe <servicename>
```

is incomplete because `kubectl describe` normally needs the resource type and name.

---

# 66. Scale a Kubernetes Deployment

The original note:

```text
kubectl scale <deploymentname> --replicasets replicastesnumber
```

Correct syntax:

```bash
kubectl scale deployment <deployment_name> --replicas=<number>
```

Example:

```bash
kubectl scale deployment my-app --replicas=3
```

This changes the desired number of Pods managed by the Deployment.

---

# 67. Update a Deployment Image

The original note:

```text
kubectl set image <deployment name>=image
```

Correct syntax:

```bash
kubectl set image deployment/<deployment_name> <container_name>=<image>
```

Example:

```bash
kubectl set image deployment/my-app nginx=nginx:1.27
```

This updates the image used by the specified container in the Deployment.

---

# 68. Delete Kubernetes Resources

The original note:

```text
kubectl delete [deployments|services|pods] --all
```

This means delete all resources of a specified type.

Examples:

```bash
kubectl delete deployments --all
```

```bash
kubectl delete services --all
```

```bash
kubectl delete pods --all
```

Be careful with:

```bash
--all
```

because it applies to all resources of that type in the current namespace.

---

# 69. Apply a Kubernetes YAML File

```bash
kubectl apply -f <k8s_yaml_file>
```

Example:

```bash
kubectl apply -f deployment.yaml
```

This tells Kubernetes to create or update the resources described in the YAML file.

This is one of the main ways Kubernetes resources are managed declaratively.

---

# 70. Quick Command Map

## Docker Information

```bash
docker info
```

## Images

```bash
docker image pull <image>
docker image ls
docker image inspect <image>
docker image rm <image>
docker build -t <image> .
docker tag <image> <new_image>
docker image push <image>
```

## Containers

```bash
docker container create ...
docker container run ...
docker container ls
docker container ls -a
docker container start ...
docker container stop ...
docker container rm ...
docker container exec ...
docker container cp ...
```

## Networking

```bash
docker network ls
docker network create <network>
docker network connect <network> <container>
docker network disconnect <network> <container>
docker network inspect <network>
```

## Storage

```bash
docker volume create <volume>
docker volume ls
docker volume inspect <volume>
```

## Registry

```bash
docker login
docker tag <image> <registry/image:tag>
docker image push <registry/image:tag>
```

## Compose

```bash
docker compose up
docker compose down
docker compose ps
```

## Swarm

```bash
docker swarm init
docker node ls
docker service create ...
docker service ls
docker service scale ...
docker service update ...
docker service rollback ...
docker service rm ...
docker stack deploy ...
docker secret create ...
```

## Kubernetes

```bash
minikube start
minikube status
minikube ip
minikube dashboard

kubectl cluster-info
kubectl get nodes
kubectl get pods
kubectl get all
kubectl create deployment ...
kubectl expose deployment ...
kubectl describe ...
kubectl scale deployment ...
kubectl set image ...
kubectl exec ...
kubectl proxy
kubectl delete ...
kubectl apply -f ...
```
