## PODS 

A Pod is the smallest deployable unit in Kubernetes and can contain one or more containers.

## Replication Controller(RC)

A replica controller is the older Kubernetes object used to ensure a specified number of Pod relicas are running at all times.
```
ReplicationController
        |
        +---- Pod 1
        +---- Pod 2
        +---- Pod 3 (Failed)
                    |
                    +--> New Pod Created
```
Commands:-

- kubectl apply -f rc.yaml
- kubectl get rc
- kubectl get pods

## ReplicaSet(RS)

ReplicaSet is the next generation of ReplicationController.A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.

ReplicaSet
     |
     +---- Pod 1
     +---- Pod 3
     |
     +---- New Pod 2 Created Automatically

kubectl apply -f rs.yaml
kubectl get rs
kubectl get pods
kubectl scale rs nginx-rs --replicas=5

## Deployment

A Deployment manages ReplicaSets and provide rolling updates and rollbacks.

Deployment
      |
      +---- ReplicaSet
                |
                +---- Pod1
                +---- Pod2
                +---- Pod3

Why Deployment?

ReplicaSet alone cannot:
❌ Rollback versions
❌ Rolling updates
❌ Revision history
Deployment provides all these features.

kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get rs
kubectl get pods

Rollback:-
kubectl rollout history deployment nginx-deployment
kubectl rollout undo deployment nginx-deployment

### Difference between ReplicationController and ReplicaSet?
Imagine you have these Pods:
Pod-1
labels:
  app: nginx
  env: prod

Pod-2
labels:
  app: nginx
  env: dev

Pod-3
labels:
  app: apache
  env: prod

ReplicationController (Equality-Based Only): “Select Pods where app equals nginx”
selector:
  app: nginx

ReplicaSet (Equality-Based) + ReplicaSet (Set-Based) : "Select Pods whose app is either nginx OR apache"

selector:
  matchExpressions:
  - key: app
    operator: In
    values:
    - nginx
    - apache

### Difference between ReplicaSet and Deployment?

ReplicaSet                      Deployment

Manages Pods                  Manages ReplicaSets

No rollback                   Rollback supported

No rolling updates            Rolling updates

Rarely created directly        Most commonly used

### Field Explanations

apiVersion: Kubernetes API version used for the resource.
kind: Type of Kubernetes resource (Deployment, Pod, Service, ReplicaSet, etc.).
metadata: Identifies the object using name, labels, namespace, etc.
spec: Desired state definition.
replicas: Number of Pod copies to run.
selector: Determines which Pods belong to the Deployment.
template: Blueprint used to create Pods.
containers: Defines container(s) running inside the Pod.
image: Container image pulled from registry.
containerPort: Port exposed by the application inside the container.

### Common Additional Fields
namespace, resources (requests/limits), env, volumeMounts, volumes, restartPolicy.

