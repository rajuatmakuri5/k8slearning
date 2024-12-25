# Kubernetes for beginners - Exercises

This repository contains a collection of **Kubernetes commands** and exercises to help you get hands-on experience with Kubernetes. The exercises cover basic operations, from creating a pod to deploying applications and services in Kubernetes.

## Common Commands

Here are some of the most commonly used `kubectl` commands for managing Kubernetes resources:

```bash
alias k=kubectl

# View the status of pods, nodes, and deployments
k get pods
k get nodes
k get pods -o wide
k get nodes -o wide
k get pods --show-labels
k get all
k get pods -n kube-system
k get deployments
k get svc
k get pvc
k get rs
k get secrets
k get endpoints

# Describe resources for detailed information
k describe pod <podname>
k explain pod.spec.name   # or `k explain pod`
k explain service
k explain deployment

```
# **Exercise 1: Creating Our First Pod**

### **Pod 1: nginx**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: first-pod-nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### **Pod 2: custom-nginx**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-app
spec:
  containers:
  - name: myfirstapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```
### **Commands**
```bash
kubectl apply -f <filename>.yaml
kubectl describe pod <podname>
kubectl exec <poname> -- ls
kubectl -it exec <podname> -- sh
wget http://localhost:80
cat index.html
kubectl delete pod nginx
```


# **Exercise2: creating our first deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
# **commands:**
```bash
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl get rs
kubectl get pods --show-labels
```
**updating(rollout):**
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
kubectl get pods
kubectl rollout status deployment/nginx-deployment
kubectl describe deployment nginx-deployment
```

**roll back an update:**
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.161 --record
kubectl rollout status deployment/nginx-deployment
kubectl get rs
kubectl get pods
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment [or] kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

# **Exercise3: ClusterIp service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
```bash
kubectl apply -f nginx-service.yaml
kubectl get services
```

**Testing the service**

**deploy a busybox pod to test the service:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    stdin: true
    tty: true
```
**Commands**
```bash
kubectl apply -f busybox-pod.yaml
kubectl exec -it busybox -- /bin/sh
test connection to service by curl/wget commands 
wget -qO- nginx-service
```
