# Kubernetes Hands-On Notes — From Deployment to Rolling Updates and Rollbacks

> Practical notes covering the Kubernetes workflow practiced with Minikube, from creating a Deployment to Scaling, Self-Healing, Services, Rolling Updates, failed rollouts, and Rollbacks.

---

## 1. Environment

The practical work was done with:

- Minikube
- Kubernetes
- Docker driver
- containerd runtime
- One-node Minikube cluster

Check the cluster:

```bash
minikube status
minikube profile list
```

Minikube provides a local Kubernetes cluster for practicing Kubernetes concepts.

---

# 2. Kubernetes Mental Model

A useful initial mental model:

> Kubernetes orchestrates containers and manages their lifecycle at a higher level.

Docker primarily builds and runs containers.

Kubernetes manages workloads and their desired state across a cluster.

The hierarchy practiced:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
    ↓
Image
```

A Service is separate from this ownership chain:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods ← Labels
          ↑
       Selector
          ↑
       Service
```

The Service selects Pods using labels.

---

# 3. Image → Container → Pod

Docker directly creates containers from images:

```text
Image
  ↓
Container
```

Kubernetes normally creates:

```text
Pod
  ↓
Container
  ↓
Image
```

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain one or more containers. In our example, each Pod contained one application container.

---

# 4. Create the First Deployment

```bash
kubectl create deployment kubernetes-bootcamp   --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

This creates a Deployment which manages a ReplicaSet, which manages Pods.

Check:

```bash
kubectl get deployments
kubectl get pods
```

Architecture:

```text
Deployment: kubernetes-bootcamp
        ↓
ReplicaSet
        ↓
Pod(s)
        ↓
kubernetes-bootcamp:v1
```

---

# 5. Deployment and ReplicaSet

The simplified control relationship is:

```text
Deployment
    ↓ manages
ReplicaSet
    ↓ maintains
Pods
```

The Deployment manages ReplicaSets.

The ReplicaSet maintains the desired number of Pods.

---

# 6. Inspect the ReplicaSet

List ReplicaSets:

```bash
kubectl get rs
```

Inspect one:

```bash
kubectl describe rs kubernetes-bootcamp-5cc66bcc9b
```

Important fields include:

```text
Selector:
    app=kubernetes-bootcamp
    pod-template-hash=5cc66bcc9b

Controlled By:
    Deployment/kubernetes-bootcamp
```

The ReplicaSet showed the desired/current/ready replica counts.

The hash is the `pod-template-hash`, identifying the Pod template associated with that ReplicaSet.

---

# 7. Labels and Selectors

Pods had labels such as:

```text
app=kubernetes-bootcamp
pod-template-hash=5cc66bcc9b
```

A selector matches resources by labels.

For example:

```text
Selector:
    app=kubernetes-bootcamp
```

means:

> Select Pods whose label contains `app=kubernetes-bootcamp`.

Services and ReplicaSets use selectors to identify the Pods they care about.

Pod names are dynamic, so Kubernetes should not rely on Pod names as stable identifiers.

---

# 8. Scaling the Deployment

We scaled the Deployment to 8 replicas:

```bash
kubectl scale deployment kubernetes-bootcamp --replicas=8
```

Then:

```bash
kubectl get pods
```

showed 8 Pods.

We did not manually create eight Pods. We changed the desired state and Kubernetes reconciled the cluster.

Conceptually:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod 1
Pod 2
Pod 3
Pod 4
Pod 5
Pod 6
Pod 7
Pod 8
```

---

# 9. Desired State and Reconciliation

Kubernetes works heavily around desired state.

If:

```text
Desired = 8
Actual  = 7
```

the controller creates another Pod.

If:

```text
Desired = 8
Actual  = 8
```

no scaling action is required.

If:

```text
Desired = 8
Actual  = 9
```

the controller removes excess Pods.

This reconciliation model is fundamental to Kubernetes.

---

# 10. Self-Healing

We tested deleting Pods.

The sequence is:

```text
Pod deleted
    ↓
ReplicaSet notices current < desired
    ↓
Replacement Pod created
    ↓
Desired replica count restored
```

Example:

```text
Before:
Desired = 8
Current = 8

Delete one Pod:

Desired = 8
Current = 7

ReplicaSet creates replacement:

Desired = 8
Current = 8
```

This demonstrates Kubernetes self-healing at the workload level.

---

# 11. Inspect a Pod

Useful command:

```bash
kubectl describe pod <pod-name>
```

Important sections:

```text
Labels
Controlled By
Containers
Image
Image ID
State
Ready
Restart Count
Events
```

Example relationship:

```text
Controlled By:
    ReplicaSet/kubernetes-bootcamp-5cc66bcc9b
```

This confirms:

```text
Pod
  ↓
ReplicaSet
```

---

# 12. Pod IPs

Use:

```bash
kubectl get pods -o wide
```

This adds information such as:

- Pod IP
- Node
- Status
- Restart count

Example:

```text
NAME                         IP            NODE
kubernetes-bootcamp-...      10.244.0.5    minikube
```

Pod IPs are not stable application endpoints.

Pods are disposable and can be recreated with different IP addresses.

This is one reason Services are needed.

---

# 13. Kubernetes Service

A Service provides a stable network endpoint for a group of Pods.

We exposed the Deployment using NodePort:

```bash
kubectl expose deployment kubernetes-bootcamp   --type=NodePort   --port=8000
```

Check:

```bash
kubectl get services
```

The Service conceptually had:

```text
Service port = 8000
NodePort     = 32198
```

---

# 14. Service Types

## ClusterIP

The default Service type.

Used for internal cluster communication.

It provides a stable internal endpoint and is commonly used in production.

## NodePort

Exposes a Service through a port on a Kubernetes node.

Conceptually:

```text
Node IP : NodePort
```

It is convenient for local Minikube experiments.

## LoadBalancer

Provides an external load-balancing mechanism when supported by the environment, commonly through cloud infrastructure.

It is not simply a synonym for "production Service"; the correct type depends on the architecture.

---

# 15. Service Port Mapping

One of the important debugging exercises was understanding:

- `port`
- `targetPort`
- `nodePort`

The Service initially targeted the wrong application port.

The correct conceptual mapping became:

```text
Browser
   ↓
NodePort 32198
   ↓
Service port 8000
   ↓
targetPort 8080
   ↓
Application
```

Therefore:

```text
NodePort 32198
      ↓
Service 8000
      ↓
Pod/Application 8080
```

---

# 16. port vs targetPort vs nodePort

## `port`

The port exposed by the Service.

Example:

```text
port: 8000
```

## `targetPort`

The port on the selected Pod/container where traffic is sent.

Example:

```text
targetPort: 8080
```

## `nodePort`

The port exposed on the Kubernetes node for a NodePort Service.

Example:

```text
nodePort: 32198
```

Remember:

```text
NodePort
   ↓
Service port
   ↓
targetPort
   ↓
Application
```

---

# 17. Debugging a Service

Inspect the Service:

```bash
kubectl describe service kubernetes-bootcamp
```

Important fields:

```text
Selector:
    app=kubernetes-bootcamp

Type:
    NodePort

ClusterIP:
    10.99.82.115

Port:
    8000/TCP

TargetPort:
    8000/TCP
```

The application was actually listening on port `8080`, so the target port needed to be changed to `8080`.

After fixing the mapping, the Service became reachable.

The Service also showed multiple endpoints, corresponding to the matching Pods.

---

# 18. Service Uses Selectors, Not Pod Names

The Service used:

```text
Selector:
    app=kubernetes-bootcamp
```

The Pods had:

```text
app=kubernetes-bootcamp
```

Therefore the Service selected those Pods.

Conceptually:

```text
Service
  |
  | selector: app=kubernetes-bootcamp
  |
  +----------+----------+
             |
       Matching Pods
```

If Pods are deleted and recreated, new Pods with matching labels can automatically become Service endpoints.

---

# 19. Observing Service Routing

Repeated requests to the Service returned different Pod names.

The application response contained information such as:

```text
Hello Kubernetes bootcamp!
Running on: kubernetes-bootcamp-...
v=1
```

Refreshing repeatedly produced different Pod names.

This demonstrated that the Service was routing traffic across multiple matching Pods.

---

# 20. Rolling Updates

A Deployment can update an image without manually deleting all Pods.

We used:

```bash
kubectl set image deployment kubernetes-bootcamp   kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v2
```

This changes the Pod template used by the Deployment.

The Deployment creates/manages a new ReplicaSet.

Before the update:

```text
Deployment
    ↓
Old ReplicaSet: 5cc66bcc9b
    ↓
v1 Pods
```

During the update:

```text
Deployment
      |
      +----------------------+
      |                      |
      v                      v
Old ReplicaSet          New ReplicaSet
5cc66bcc9b              55d75dfbd9
      |                      |
      v                      v
v1 Pods                  v2 Pods
```

This is the basic idea of a RollingUpdate.

---

# 21. Failed Rolling Update

The original image:

```text
gcr.io/google-samples/kubernetes-bootcamp:v2
```

could not be pulled in our environment.

New Pods entered:

```text
ErrImagePull
```

and then:

```text
ImagePullBackOff
```

Old Pods were still running.

Conceptually:

```text
Deployment
      |
      +-------------------------+
      |                         |
      v                         v
Old ReplicaSet             New ReplicaSet
     v1                         v2
     |                          |
     v                          v
Running Pods              ImagePullBackOff
     ✅                         ❌
```

This demonstrated an important property of rolling updates:

> A failed new version does not necessarily mean all old Pods are immediately removed.

The old ReplicaSet can remain available while the new Pods fail to become ready.

---

# 22. ErrImagePull vs ImagePullBackOff

## ErrImagePull

Kubernetes attempted to pull the image and encountered an error.

## ImagePullBackOff

Kubernetes is backing off before retrying the image pull.

Typical sequence:

```text
Image pull fails
      ↓
ErrImagePull
      ↓
Retries
      ↓
ImagePullBackOff
```

Diagnose the exact reason with:

```bash
kubectl describe pod <pod-name>
```

Then inspect:

```text
Events
```

The Events section normally contains the specific image-pull error.

---

# 23. Rollout History

View Deployment revisions:

```bash
kubectl rollout history deployment/kubernetes-bootcamp
```

We initially saw:

```text
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

After rollback:

```text
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

After another update:

```text
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>
```

`CHANGE-CAUSE` was `<none>` because no change-cause metadata had been recorded.

---

# 24. Rollback

We rolled back the failed rollout with:

```bash
kubectl rollout undo deployment/kubernetes-bootcamp
```

The Deployment returned to the previous working Pod template.

The old ReplicaSet became active again and replacement Pods were created as necessary.

Conceptually:

```text
Failed v2 rollout
      ↓
rollout undo
      ↓
Previous working Pod template
      ↓
Working Pods
```

---

# 25. Rollback Does Not Mean Reusing the Exact Same Pods

Rollback restores the Deployment's previous Pod template/configuration.

It does not necessarily resurrect the exact old Pod objects.

For example, new Pods can have new names while belonging to the old ReplicaSet:

```text
kubernetes-bootcamp-5cc66bcc9b-f56zq
kubernetes-bootcamp-5cc66bcc9b-hlpkm
```

The important distinction is:

```text
Same ReplicaSet / Pod template
        ≠
Same individual Pod object
```

Pods are disposable.

---

# 26. Rollback to a Specific Revision

General syntax:

```bash
kubectl rollout undo deployment/<deployment-name>   --to-revision=<revision-number>
```

For this Deployment:

```bash
kubectl rollout undo deployment/kubernetes-bootcamp   --to-revision=<revision-number>
```

Difference:

```text
kubectl rollout undo
```

→ Undo to the previous revision.

```text
kubectl rollout undo ... --to-revision=N
```

→ Restore a specific revision.

---

# 27. Successful v2 Update

Because the original `gcr.io/...:v2` image was unavailable in our environment, we used:

```text
wallymathieu/kubernetes-bootcamp:v2
```

The Deployment was updated to that image.

Then:

```bash
kubectl get pods -o wide
```

showed all 8 Pods as:

```text
1/1 Running
```

The new ReplicaSet hash was:

```text
698cb4ffcc
```

---

# 28. Verify the Actual Image

Use:

```bash
kubectl describe pod <pod-name>
```

Look under:

```text
Containers
```

and inspect:

```text
Image:
Image ID:
State:
Ready:
```

The successful Pods showed:

```text
Image:
    wallymathieu/kubernetes-bootcamp:v2

State:
    Running

Ready:
    True
```

This verifies that the Pods are actually running the new image.

---

# 29. ReplicaSet Hashes During Updates

The ReplicaSet hash changed as the Pod template changed.

Examples from the practical work:

```text
v1
↓
ReplicaSet 5cc66bcc9b
```

Failed v2 attempt:

```text
v2
↓
ReplicaSet 55d75dfbd9
```

Successful v2:

```text
v2
↓
ReplicaSet 698cb4ffcc
```

The hash is the `pod-template-hash` associated with the Pod template.

Different Pod templates result in different ReplicaSets.

---

# 30. Useful Rollout Commands

View rollout status:

```bash
kubectl rollout status deployment/<deployment-name>
```

View history:

```bash
kubectl rollout history deployment/<deployment-name>
```

Undo previous rollout:

```bash
kubectl rollout undo deployment/<deployment-name>
```

Undo to a specific revision:

```bash
kubectl rollout undo deployment/<deployment-name>   --to-revision=<N>
```

Update image:

```bash
kubectl set image deployment/<deployment-name>   <container-name>=<image>
```

---

# 31. Useful Inspection Commands

Deployments:

```bash
kubectl get deployments
kubectl describe deployment <name>
```

Pods:

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

ReplicaSets:

```bash
kubectl get rs
kubectl describe rs <name>
```

Services:

```bash
kubectl get services
kubectl describe service <name>
```

Minikube:

```bash
minikube status
minikube profile list
minikube service <service-name> --url
```

---

# 32. Complete Practical Timeline

## Step 1 — Create Deployment

```bash
kubectl create deployment kubernetes-bootcamp   --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

Result:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
    ↓
v1 Image
```

## Step 2 — Scale

```bash
kubectl scale deployment kubernetes-bootcamp --replicas=8
```

Result:

```text
8 Pods
```

## Step 3 — Test Self-Healing

Delete a Pod.

ReplicaSet notices:

```text
Current < Desired
```

and creates a replacement.

## Step 4 — Create Service

```bash
kubectl expose deployment kubernetes-bootcamp   --type=NodePort   --port=8000
```

## Step 5 — Debug Port Mapping

The application listened on:

```text
8080
```

The Service initially targeted:

```text
8000
```

Correct path:

```text
NodePort 32198
      ↓
Service 8000
      ↓
targetPort 8080
      ↓
Application
```

## Step 6 — Observe Service Routing

Repeated requests returned different Pod names.

## Step 7 — Attempt Rolling Update

```bash
kubectl set image deployment kubernetes-bootcamp   kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v2
```

New Pods failed with:

```text
ErrImagePull
ImagePullBackOff
```

## Step 8 — Inspect History

```bash
kubectl rollout history deployment/kubernetes-bootcamp
```

## Step 9 — Roll Back

```bash
kubectl rollout undo deployment/kubernetes-bootcamp
```

## Step 10 — Use a Working v2 Image

```text
wallymathieu/kubernetes-bootcamp:v2
```

## Step 11 — Verify

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

Result:

```text
1/1 Running
Image: wallymathieu/kubernetes-bootcamp:v2
```

---

# 33. Final Architecture

```text
                         Deployment
                    kubernetes-bootcamp
                              |
                              v
                     ReplicaSet 698cb4ffcc
                              |
              +---------------+---------------+
              |       |       |       |       |
              v       v       v       v       v
            Pod     Pod     Pod     Pod     Pod
              |       |       |       |       |
              +-------+-------+-------+-------+
                              |
                    label: app=kubernetes-bootcamp
                              ^
                              |
                         Service Selector
                              |
                         Kubernetes Service
                              |
                           NodePort
                              |
                           Browser
```

Application image at the end:

```text
wallymathieu/kubernetes-bootcamp:v2
```

---

# 34. Core Concepts Learned

- Minikube
- Kubernetes cluster
- Deployment
- ReplicaSet
- Pod
- Container
- Image
- Labels
- Selectors
- Desired state
- Reconciliation
- Scaling
- Self-healing
- Pod IPs
- Service
- ClusterIP
- NodePort
- LoadBalancer
- Service `port`
- Service `targetPort`
- Service `nodePort`
- Service endpoints
- RollingUpdate
- ReplicaSet replacement
- Image pull failures
- `ErrImagePull`
- `ImagePullBackOff`
- Rollout history
- Deployment revisions
- Rollback
- Rollback to a specific revision
- Pod-template hashes

---

# 35. The Most Important Mental Model

Do not memorize Kubernetes commands in isolation.

Understand what Kubernetes is doing:

```text
You declare the desired state.
            ↓
Kubernetes compares
desired state vs actual state.
            ↓
Controllers reconcile the difference.
```

For our practical example:

```text
"Give me 8 replicas"
        ↓
ReplicaSet maintains 8 Pods.

"Expose these Pods"
        ↓
Service selects them by labels.

"Change the application image"
        ↓
Deployment creates/manages a new ReplicaSet.

"New image cannot be pulled"
        ↓
New Pods fail.
Old Pods remain during the rollout.

"Undo"
        ↓
Deployment returns to a previous Pod template.
```

That control-loop model is more important than memorizing individual commands.
