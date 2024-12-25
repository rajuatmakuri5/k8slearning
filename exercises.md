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

# **Exercise4: Deamonsets**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
        - name: fluentd-elasticsearch
          image: quay.io/fluentd_elasticsearch/fluentd:latest
```
**Commands:**
```bash
kubectl apply -f fluentd.yaml
kubectl get pods -o wide
kubectl get daemonset
```
**Using node selector to define scope**
``` yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      nodeSelector:
        kubernetes.io/hostname: node01
      containers:
        - name: fluentd-elasticsearch
          image: quay.io/fluentd_elasticsearch/fluentd:latest
```
**Commands:**
```bash
kubectl apply -f fluentd.yaml
kubectl get pods -o wide
kubectl get daemonset
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      nodeSelector:
        disk: ssd
      containers:
        - name: fluentd-elasticsearch
          image: quay.io/fluentd_elasticsearch/fluentd:latest
```
**Commands:**
```bash
kubectl label node node02 disk=ssd
kubectl apply -f fluentd.yaml
kubectl get pods -o wide
kubectl get daemonset
```
# **Exercise 5: Kubernetes Storage**
**Create local storage class**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
```
kubectl apply -f local-sc.yaml
kubectl get sc
```
**Create a Persistent Volume (PV):**
We need to create local directory /mnt/data on node01 -r node02 
This configuration will create a PersistentVolume that uses /mnt/data as the local storage
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node01
```
```
kubectl apply -f local-pv.yaml
```
**Create PersistentVolumeClaim(PVC):**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
```
```
kubectl apply -f local-pvc.yaml
kubectl get storageclass
kubectl get pv
kubectl get pvc
```
**Attach the storage to a test pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/data"
          name: local-storage
  volumes:
    - name: local-storage
      persistentVolumeClaim:
        claimName: local-pvc
```
```
kubectl apply test-pod.yaml
kubectl get pods
```
Login to the test pod and lets create few files:
```
k exec -it test-pod -- sh
/ # cd  data/
/data # ls -lrt
total 0
/data # touch test1
/data # touch test2
/data # touch test3
/data # ls -lrt
total 0
-rw-r--r--    1 root     root             0 Dec 25 10:42 test1
-rw-r--r--    1 root     root             0 Dec 25 10:42 test2
-rw-r--r--    1 root     root             0 Dec 25 10:42 test3
/data # exit
```
Delete and recreate the pod to see if the files are still exist
```
k delete pod test-pod
k get pods
k apply -f test-pod.yaml
k exec -it test-pod -- sh
cd data/
ls -lrt
```
We can alos verify the local directory on the node:
```
cd /mnt/data 
```


