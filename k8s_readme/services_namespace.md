## What is Service in Kubernetes
A Kubernetes Service is an object that provides a stable network endpoint for accessing a group of Pods. Since Pods are temporary and their IP addresses change when they are recreated, a Service gives applications a consistent way to communicate.

## Why do we need a Service?
Imagine you have three Pods running a web application:
- web-app-pod-1  → 10.244.1.5
- web-app-pod-2  → 10.244.1.6
- web-app-pod-3  → 10.244.1.7

If one Pod crashes:
- web-app-pod-2 dies
- New Pod starts → 10.244.1.20

The IP changed. Without a Service, clients would need to keep track of changing Pod IPs. A Service solves this by providing a fixed virtual IP and DNS name.The Service automatically routes traffic to healthy Pods.
```
Clients
   |
   v
Service (10.96.0.10)
   |
   +--> Pod 1
   +--> Pod 2
   +--> Pod 3
```
## How a Service works.
- 1. Pods are created with labels.
- 2. The Service uses a selector to find matching Pods.
- 3. Kubernetes creates a virtual IP (ClusterIP).
- 4. Traffic sent to the Service is load-balanced across the Pods.

## Types of Kubernetes Services
1. ClusterIP (Default)
- * Accessible only within the cluster.
- * Used for communication between applications.
```
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

2. NodePort Service
- A NodePort exposes your application on a specific port of every Kubernetes node.
  
When you create a NodePort Service, Kubernetes:
- * Creates a ClusterIP (internal access).
- * Opens the same port (typically 30000–32767) on every node.
- * Forwards traffic from that node port to the appropriate Pods.
```
Internet
    |
http://192.168.1.10:30080
    |
+-------------------------+
| Kubernetes Node         |
| NodePort: 30080         |
+-------------------------+
          |
      ClusterIP Service
          |
    +-----+-----+
    |           |
 Pod 1       Pod 2
```
If your cluster has three nodes:
- You can access the application through any node’s IP using the NodePort.
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```
Access it with: http://<Node-IP>:30080

Advantages

- * Simple to configure.
- * Works on any Kubernetes cluster.
- * Useful for development and testing.

## LoadBalancer Service

A LoadBalancer Service automatically provisions an external load balancer (on supported cloud platforms) that provides a single public IP or DNS name for your application.Your loadbalancer service will act as nodeport if you are not using any managed cloud Kubernetes such as GKE,AKS,EKS etc.

The cloud load balancer distributes incoming requests across healthy Pods.

```
Internet
     |
Public IP
     |
Cloud Load Balancer
     |
Kubernetes Service
     |
 +----+----+
 |         |
Pod 1    Pod 2
```
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

## Kubernetes Namespace
A Kubernetes Namespace is a logical partition inside a Kubernetes cluster. It helps organize resources, isolate teams or applications, and avoid naming conflicts.

Think of a namespace like folders on your computer.

- * Your computer = Kubernetes Cluster
- * Folder = Namespace
- * Files = Kubernetes resources (Pods, Deployments, Services, ConfigMaps, Secrets, etc.)

-Without folders, all files would be in one place. Similarly, without namespaces, all Kubernetes resources would exist together in the default namespace.

## Why do we need Namespaces?
Suppose your company has three teams:

- * Development
- * Testing (QA)
- * Production

- Each team wants to deploy an Nginx application named nginx.
- Kubernetes doesn’t allow multiple Deployments with the same name in the same namespace.
- Each namespace has its own set of resources, so duplicate names are allowed across namespaces.
```
Cluster

Development Namespace
 └── nginx

Testing Namespace
 └── nginx

Production Namespace
 └── nginx
```
## Default Namespaces
When you create a Kubernetes cluster, several namespaces already exist.

1. default:-
- If you don’t specify a namespace, Kubernetes creates resources here.

2. kube-system:-
- Contains Kubernetes system components such as:
- Never delete resources here unless you know exactly what you’re doing.
- * DNS
- * Scheduler
- * Controller Manager
- * kube-proxy

3. kube-public:-
- Stores information readable by all users. Rarely used in day-to-day work.

4. kube-node-lease:-
- Used by nodes to send heartbeats to the control plane.

## Create a Namespace
```
kubectl create namespace dev
or
apiVersion: v1
kind: Namespace
metadata:
  name: dev

kubectl apply -f namespace.yaml
```
## View Namespaces
```
kubectl get namespaces
```
## Create Resources Inside a Namespace
```
kubectl create deployment nginx --image=nginx  -n dev

kubectl get deployments -n dev
```
## List Resources in a Namespace

1. Pods:
```
kubectl get pods -n dev
```

2. Services:
```
kubectl get svc -n dev
```
3. Deployments:
```
kubectl get deployment -n dev
```
4. Everything:
```
kubectl get all -n dev
```
## Switch the Default Namespace

Instead of typing -n dev every time:
```
kubectl config set-context --current --namespace=dev
Check:-
kubectl config view --minify
```
Return to the default namespace:
```
kubectl config set-context --current --namespace=default
```
## Communication Between Namespaces
Suppose you have:

- Namespace: dev
- Service: nginx

and

- Namespace: qa
- Pod: test

The Pod in qa cannot simply use:
- http://nginx

Instead, use the fully qualified DNS name:
- http://nginx.dev.svc.cluster.local

Format:
- service-name.namespace.svc.cluster.local

## Namespace Isolation

Namespaces isolate:

- * Pods
- * Deployments
- * Services
- * Secrets
- * ConfigMaps
- * Jobs
- * ReplicaSets

They do not isolate:

- * Nodes
- * PersistentVolumes (PV)
- * StorageClasses
- * ClusterRoles
- * ClusterRoleBindings
- * CustomResourceDefinitions (CRDs)

## Resource Quotas
You can limit CPU, memory, and object counts within a namespace.
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
```
## Limit Ranges

You can define default or maximum CPU and memory requests for containers.
```
apiVersion: v1
kind: LimitRange
metadata:
  name: limits
  namespace: dev
```
## Delete a Namespace
```
kubectl delete namespace dev
```
Warning: Deleting a namespace deletes almost all resources inside it.

## Common Commands
```
kubectl get ns
kubectl describe namespace dev
kubectl create namespace qa
kubectl delete namespace qa
kubectl get pods -A
kubectl get pods -n dev
kubectl get all -n dev
kubectl config set-context --current --namespace=dev
```
