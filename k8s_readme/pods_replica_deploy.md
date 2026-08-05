## PODS 

A Pod is the smallest deployable unit in Kubernetes and can contain one or more containers.

## Kubernetes Pod: Life Cycle

1. Pending - The pod has been accepted by the API server, but not yet running.
Why it happens:
- Scheduler hasn’t assigned a node yet
- Image is still being pulled
- Node doesn’t have enough CPU/memory
- PVC is still being attached
How to check:
run kubectl describe pod <pod>
look for :-
- FailedScheduling
- ImagePullBackOff
- ErrImagePull

2. Running - The pod is scheduled, containers are created, and at least one container is running.
   
4. Succeeded - The pod completed successfully and exited with code 0.
- jobs,cron jobs
- A pod running a backup script finishes → status becomes Succeeded.

5. Failed - The pod terminated with a non‑zero exit code.
Common causes:
- Application crash
- Misconfiguration
- Bad environment variables
- Missing files
- Failed init container
Debug:
```
kubectl logs <pod> --previous
kubectl describe pod <pod>
```
5. Unknown - The node cannot be reached.
Causes:
- Node is down
- Network partition
- Kubelet crash
- Cloud provider issue
- This is rare but dangerous in production.

## Pod Lifecycle Events (Important for Debugging)
Pods also go through container-level states, which matter more during troubleshooting:
- ContainerCreating - Image is being pulled, volumes attached.
- CrashLoopBackOff - Container keeps crashing → Kubernetes retries with exponential backoff.
- ImagePullBackOff - Image cannot be pulled.
- Terminating - Pod is shutting down gracefully.
- Evicted - Node pressure (CPU, memory, disk) forced the pod out.

## Pod Termination Process (Graceful Shutdown)
When a pod is deleted:
- SIGTERM sent to containers
- Kubernetes waits for terminationGracePeriodSeconds (default: 30s)
- If still running → SIGKILL
- Pod removed from API server
Why this matters:
- If your app doesn’t handle SIGTERM, you get:
- Broken connections
- Lost requests
- Corrupted data

## Init Containers (Pre‑Lifecycle Step)
Init containers run before main containers.
Used for:
- Setting up config
- Waiting for dependencies
- Preparing volumes
- If an init container fails → pod never enters Running.

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

- kubectl apply -f rc.yaml (to apply/deploy )
- kubectl explain rc (to check rc details)
- kubectl get rc (check the running rc)
- kubectl get po - check the running pods/replica
- kubectl describe po rc_name - detailed sepc of rc

## ReplicaSet(RS)

ReplicaSet is the next generation of ReplicationController.A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.
```
ReplicaSet
     |
     +---- Pod 1
     +---- Pod 3
     |
     +---- New Pod 2 Created Automatically
```
Commands:-

- kubectl apply -f rs.yaml
- kubectl explain rs (to check rc details)
- kubectl get po (check the running pods/replica
- kubectl describe po rs_name (detailed sepc of rs)
- kubectl get rs (check the running rs)
- kubectl get pods (check the running rs)

update/edit scale:-

- kubectl edit rs/rs_name (edit or update rs on live object)
- kubectl scale --replicas=10 rs/rs_name (update through command)
- we can edit directly to yaml file


## Deployment

A Deployment manages ReplicaSets and provide rolling updates and rollbacks.

```
Deployment
      |
      +---- ReplicaSet
                |
                +---- Pod1
                +---- Pod2
                +---- Pod3
```
Why Deployment?

ReplicaSet alone cannot:
- ❌ Rollback versions
- ❌ Rolling updates
- ❌ Revision history
- Deployment provides all these features.

Commands:-

- kubectl apply -f deployment.yaml (to apply)
- kubectl get deployments (check running deployments)
- kubectl explain deploy (to check deploy details)
- kubectl describe po deploy_name (detailed sepc of deployments)

Rollout:-

- kubectl get deployment nginx-deploy -o yaml (check actual container name)
- kubectl set image deployment/nginx-deploy container_name=nginx:1.26
- kubectl rollout status deployment/nginx-deploy

Rollback:-
- kubectl rollout history deploy/deloy_name (see the rollout history)
- kubectl undo deploy/deloy_name (rollback changes)

remove the old ReplicaSet:-

- kubectl rollout history deployment nginx-deploy --revision=<number>
- kubectl delete rs <old-rs-name>

### Difference between ReplicationController and ReplicaSet?

Imagine you have these Pods:
```
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
```
ReplicationController (Equality-Based Only): “Select Pods where app equals nginx”
```
selector:
  app: nginx
```
ReplicaSet (Equality-Based) + ReplicaSet (Set-Based) : "Select Pods whose app is either nginx OR apache"
```
selector:
  matchExpressions:
  - key: app
    operator: In
    values:
    - nginx
    - apache
```

### Difference between ReplicaSet and Deployment?
```
ReplicaSet                      Deployment

Manages Pods                  Manages ReplicaSets

No rollback                   Rollback supported

No rolling updates            Rolling updates

Rarely created directly        Most commonly used
```
### Dry Run

- kubectl create deploy deploy/deploy_name --image=nginx --dry-run=client (it will not deploy,for test)

Generate yaml and move to new file:-

- kubectl create deploy deploy/deploy_name --image=nginx --dry-run=clinent -o yaml > deploy.yaml

### Field Explanations

- apiVersion: Kubernetes API version used for the resource.
- kind: Type of Kubernetes resource (Deployment, Pod, Service, ReplicaSet, etc.).
- metadata: Identifies the object using name, labels, namespace, etc.
- spec: Desired state definition.
- replicas: Number of Pod copies to run.
- selector: Determines which Pods belong to the Deployment.
- template: Blueprint used to create Pods.
- containers: Defines container(s) running inside the Pod.
- image: Container image pulled from registry.
- containerPort: Port exposed by the application inside the container.

### Common Additional Fields
- namespace, resources (requests/limits), env, volumeMounts, volumes, restartPolicy.

### Rollout Strategies: Recreate vs RollingUpdate

Recreate Strategy
What it does
- Deletes all old pods at once
- Then creates new pods
- Causes downtime

When used in production
Rarely — only when:
- The app cannot run two versions at the same time
- Database schema changes require exclusive access
- Old and new versions conflict

Spec
```
strategy:
  type: Recreate
```
RollingUpdate Strategy (default)
What it does
- Gradually replaces old pods with new pods
- Ensures zero downtime
- Uses maxSurge and maxUnavailable to control speed

When used in production
- Almost always — safe, stable, predictable.

Spec
```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```
### maxSurge & maxUnavailable Tuning
maxSurge
- How many extra pods Kubernetes can create above the desired count.
- Example:
- Desired replicas: 5
- maxSurge: 2
- → Kubernetes may temporarily run 7 pods
Use cases
- High‑traffic apps needing extra capacity during rollout
- Canary‑style rollouts

maxUnavailable
- How many pods can be unavailable during rollout.
- Example:
- Desired replicas: 5
- maxUnavailable: 1
- → Kubernetes ensures at least 4 pods stay running
Use cases
- Critical apps where downtime is unacceptable
- Slow rollouts for stability

### Production Tuning Examples

High availability (banking, payments)
```
maxSurge: 1
maxUnavailable: 0
```
Fast rollout (internal apps)
```
maxSurge: 3
maxUnavailable: 2
```
Resource‑constrained clusters
```
maxSurge: 0
maxUnavailable: 1
```
### Pausing & Resuming Rollouts
Pause a rollout
- Useful when:
- You want to inspect the new ReplicaSet
- You want to apply multiple changes before resuming
- You want to stop a bad rollout mid‑way
  
```
kubectl rollout pause deployment nginx-deploy
```

Resume a rollout

```
kubectl rollout resume deployment nginx-deploy
```

Production use case
- You push a new version → metrics spike → pause rollout → investigate → resume or rollback.
- 
### Rollback Best Practices
- Always check rollout history
- Never delete old ReplicaSets manually
- Use readiness probes to avoid rolling out broken pods
- Monitor metrics during rollout
- Use kubectl describe to inspect failures
  
### Health Checks During Rollout (Readiness Probes)
Readiness probes decide when a pod is ready to receive traffic.
If readiness fails:
- Pod stays out of service
- Deployment rollout pauses automatically
- No traffic goes to broken pods
  
Why readiness probes matter in rollouts:-

1. Prevent bad versions from going live
- If the new pod fails readiness:
- Old pods stay running
- Rollout stops
- No downtime

2. Protect users from broken deployments-Traffic only goes to healthy pods.
   
3. Enable safe automated rollbacks-Some platforms (Argo Rollouts, Flagger) rollback automatically when readiness fails.

