
## 1. Kubernetes

Kubernetes is a container orchestration platform used to manage containerized applications.

Docker mainly builds and runs containers.

Kubernetes manages containerized applications by handling:

- Pods
- Deployments
- Scaling
- Networking
- Service discovery
- Application configuration
- Restarts and desired state

### Basic mental model

Cluster → Node → Pod → Container

---

## 2. Minikube

Minikube provides a local Kubernetes cluster for learning and testing.

Start the cluster:

kubectl start

Check Minikube:

minikube status

Start Minikube:

minikube start

Check the cluster:

kubectl get nodes

Example:

NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1.37.0

Important:

Minikube is our local learning environment.

If Minikube is stopped, kubectl cannot communicate with the Kubernetes API server.

---

## 3. Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain one or more containers.

For simple applications, normally:

Pod → 1 container

Multiple containers can exist in the same Pod when they need to work closely together.

Example:

Pod
└── nginx container

Create a Pod:

kubectl run nginx --image=nginx

Check Pods:

kubectl get pods

Detailed information:

kubectl describe pod nginx

Basic Pod flow:

Pod
└── Container

Important:

A Pod is not the same thing as a container.

The Pod is the Kubernetes unit that contains the container(s).

---

## 4. Deployment

A Deployment manages Pods and maintains the desired number of replicas.

Instead of manually creating individual Pods, we normally use a Deployment for applications.

Create a Deployment:

kubectl create deployment nginx --image=nginx

Check Deployments:

kubectl get deployments

Check Pods:

kubectl get pods

Basic relationship:

Deployment
    ↓
Pods
    ↓
Containers

If a Pod is deleted, the Deployment can create a replacement Pod to maintain the desired state.

---

## 5. Scaling

A Deployment can run multiple replicas of an application.

Scale nginx to 3 replicas:

kubectl scale deployment nginx --replicas=3

Check:

kubectl get pods

Example:

nginx Pod 1
nginx Pod 2
nginx Pod 3

Check Deployment:

kubectl get deployments

Example:

NAME    READY
nginx   3/3

Mental model:

Desired state:
"I want 3 nginx Pods."

Kubernetes:
"Make sure 3 nginx Pods are running."

---

## 6. Service

Pods are not a stable way to access an application because Pods can be recreated and their IP addresses can change.

A Service provides a stable way to access a group of Pods.

Basic flow:

User/Application
      ↓
Service
      ↓
Pods
      ↓
Containers

Create a NodePort Service:

kubectl expose deployment nginx --type=NodePort --port=80

Check Services:

kubectl get services

Example:

nginx   NodePort   10.110.61.194   <none>   80:30659/TCP

Access the Minikube Service:

minikube service nginx

---

## 7. Labels

A Label is a tag attached to a Kubernetes object.

Example:

app=nginx

Check Pod labels:

kubectl get pods --show-labels

Example:

nginx-xxxxx   app=nginx,pod-template-hash=xxxxx

Labels help Kubernetes identify and organize objects.

Mental model:

Label = "What are you?"

---

## 8. Selectors

A Selector is used to find objects based on their labels.

Example:

selector:
  app=nginx

If Pods have:

app=nginx

and a Service has:

selector:
  app=nginx

the Service can find those Pods.

Mental model:

Label = tag

Selector = filter

Easy way to remember:

Labels identify.
Selectors select.

---

## 9. Service Endpoints

A Service uses its selector to find matching Pods.

Example:

Service:

Selector: app=nginx

Matching Pods:

Pod 1 → app=nginx
Pod 2 → app=nginx
Pod 3 → app=nginx

The Service creates endpoints for those Pods.

Check Service details:

kubectl describe service nginx

Important fields:

Selector
TargetPort
NodePort
Endpoints

Example:

Selector: app=nginx

Endpoints:
10.244.0.2:80
10.244.0.5:80
10.244.0.3:80

We do not manually provide these Pod IPs.

Kubernetes discovers matching Pods using labels and selectors.

---

## 10. Service Types

### ClusterIP

ClusterIP provides internal access to a Service.

Example:

Service
    ↓
Pods

It is normally used for communication inside the Kubernetes cluster.

Change Service to ClusterIP:

kubectl edit service nginx

Change:

type: NodePort

to:

type: ClusterIP

Check:

kubectl get service nginx

Example:

nginx   ClusterIP   10.110.61.194   <none>   80/TCP

ClusterIP is internal to the cluster.

---

### NodePort

NodePort exposes a Service through a port on a Kubernetes node.

Flow:

User
  ↓
NodeIP:NodePort
  ↓
Service
  ↓
Pods

Example:

80:30659/TCP

80 = Service port

30659 = NodePort

NodePort is useful for learning and simple external access.

---

### LoadBalancer

LoadBalancer is used when an external/cloud load balancer should expose the Service.

Typical cloud flow:

User
  ↓
Cloud Load Balancer
  ↓
Service
  ↓
Pods

In Minikube, the EXTERNAL-IP can remain:

<pending>

because Minikube is running locally and does not automatically have a cloud provider load balancer.

Example:

nginx   LoadBalancer   10.110.61.194   <pending>   80:31563/TCP

Important:

ClusterIP → internal access

NodePort → access through a node port

LoadBalancer → external/cloud load balancer

---

## 11. Testing a ClusterIP Service

Because ClusterIP is internal, we tested the Service from inside the Kubernetes cluster.

Create a temporary Pod:

kubectl run test-pod --image=busybox:1.36 --restart=Never --rm -it -- sh

Inside the Pod:

wget -qO- http://nginx

Here:

nginx = Service name

The request flow is:

test-pod
   ↓
nginx Service
   ↓
nginx Pod
   ↓
nginx container

This demonstrated that the ClusterIP Service can be accessed from inside the cluster.

---

# ConfigMap

## 12. What is ConfigMap?

A ConfigMap stores non-sensitive application configuration separately from the application image.

Examples:

APP_ENV=production

APP_PORT=5000

LOG_LEVEL=info

Mental model:

Application
    ↓
Configuration
    ↓
ConfigMap

ConfigMap = non-sensitive configuration.

Do not use ConfigMap for passwords, API keys, or other sensitive values.

---

## 13. Create a ConfigMap

Command pattern:

kubectl create configmap <name> --from-literal=<KEY>=<VALUE>

Example:

kubectl create configmap app-config \
  --from-literal=APP_ENV=development \
  --from-literal=APP_VERSION=1.0

The backslash (\) is only Linux shell line-continuation syntax.

The same command can be written on one line:

kubectl create configmap app-config --from-literal=APP_ENV=development --from-literal=APP_VERSION=1.0

The important thing is to understand the pattern instead of memorizing the complete command.

---

## 14. Verify ConfigMap

List ConfigMaps:

kubectl get configmap

Check a specific ConfigMap:

kubectl get configmap app-config

View details:

kubectl describe configmap app-config

Example data:

APP_ENV=development

APP_VERSION=1.0

---

## 15. Using ConfigMap in a Pod

A Pod can reference a ConfigMap and make its values available as environment variables.

Example:

envFrom:
  - configMapRef:
      name: app-config

Concept:

ConfigMap
    ↓
Pod YAML
    ↓
envFrom
    ↓
Container environment variables

We created a learning Pod using:

config-test.yaml

and applied it with:

kubectl apply -f config-test.yaml

Check the Pod:

kubectl get pod config-test

Verify environment variables:

kubectl exec config-test -- env | grep APP_

Example:

APP_ENV=development
APP_VERSION=1.0

---

# Secret

## 16. What is a Kubernetes Secret?

A Kubernetes Secret is used to store sensitive application information.

Examples:

- Passwords
- API keys
- Tokens
- Credentials

Mental model:

ConfigMap → normal configuration

Secret → sensitive configuration

Important:

Kubernetes Secrets are not automatically "secure" just because they are called Secrets. Proper access control and cluster security are still required.

---

## 17. Create a Secret

Command pattern:

kubectl create secret generic <name> --from-literal=<KEY>=<VALUE>

Example:

kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=mysecret123

Check the Secret:

kubectl get secret app-secret

Example:

NAME         TYPE     DATA   AGE
app-secret   Opaque   1      ...

Opaque is the normal type for a generic application Secret.

---

## 18. Inspect a Secret

Describe the Secret:

kubectl describe secret app-secret

Kubernetes shows the size of the value rather than displaying the actual password.

Example:

DB_PASSWORD: 11 bytes

Do not expect the actual secret value to appear in normal describe output.

---

## 19. Using Secret in a Pod

A Pod can reference a Secret:

envFrom:
  - secretRef:
      name: app-secret

Example:

secret-test.yaml

Apply:

kubectl apply -f secret-test.yaml

Check:

kubectl get pod secret-test

Verify that the container received the environment variable:

kubectl exec secret-test -- env | grep DB_PASSWORD

Expected:

DB_PASSWORD=mysecret123

Flow:

Kubernetes Secret
       ↓
secretRef
       ↓
Pod
       ↓
Container environment
       ↓
DB_PASSWORD

---

# ConfigMap vs Secret

ConfigMap:

→ Non-sensitive configuration

Examples:

APP_ENV=production
APP_VERSION=1.0
LOG_LEVEL=info

Secret:

→ Sensitive configuration

Examples:

DB_PASSWORD
API_KEY
ACCESS_TOKEN

Important distinction:

GitHub Secret
→ Used by GitHub Actions / CI/CD

Kubernetes Secret
→ Used by applications running inside Kubernetes

They are different systems and solve different problems.

---

# Kubernetes YAML and Real Projects

In a real Kubernetes project, resources are normally declared in YAML files and stored in Git.

Example structure:

project/
├── deployment.yaml
├── service.yaml
├── configmap.yaml
└── secret.yaml

Not every project needs every resource.

Choose resources based on application requirements.

Examples:

Need multiple application replicas
→ Deployment

Need stable access to Pods
→ Service

Need non-sensitive configuration
→ ConfigMap

Need sensitive configuration
→ Secret

Need data to survive Pod replacement
→ Persistent Storage

Need HTTP/HTTPS routing
→ Ingress

Mental model:

Application requirement
        ↓
Choose Kubernetes resource
        ↓
Create YAML
        ↓
kubectl apply
        ↓
Verify
        ↓
Troubleshoot

---

# Important Commands

These are common commands to become familiar with:

kubectl get nodes

kubectl get pods

kubectl get deployments

kubectl get services

kubectl get configmap

kubectl get secret

kubectl describe pod <name>

kubectl describe service <name>

kubectl describe configmap <name>

kubectl describe secret <name>

kubectl logs <pod>

kubectl exec <pod> -- <command>

kubectl apply -f <file>

kubectl scale deployment <name> --replicas=<number>

---

# What NOT to Memorize

Do not try to memorize every Kubernetes command or every YAML field.

Focus on:

1. Understanding the Kubernetes object.
2. Understanding what problem it solves.
3. Knowing the common command patterns.
4. Being able to read YAML.
5. Knowing where to check the official documentation when syntax is forgotten.
6. Being able to troubleshoot when something does not work.

The goal is:

Understand the requirement
        ↓
Choose the Kubernetes resource
        ↓
Create/read the YAML
        ↓
Apply it
        ↓
Verify the result
        ↓
Troubleshoot if needed

---
