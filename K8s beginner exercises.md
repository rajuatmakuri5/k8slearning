**Exercise1: creating our first pod:**

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

**commands**:
kubectl apply -f myfirstpod.yaml
kubectl describe pod myfirstapp
kubectl exec myfirstapp -- ls
kubectl -it exec myfirstapp -- sh
wget http://localhost:80
cat index.html
kubectl delete pod nginx
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
**Exercise2: creating our first deployment**

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

**commands:**
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl get rs
kubectl get pods --show-labels

updating(rollout):
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
kubectl get pods
kubectl rollout status deployment/nginx-deployment
kubectl describe deployment nginx-deployment

roll back an update:
kubectl set image deployment/nginx-deployment nginx=nginx:1.161
kubectl rollout status deployment/nginx-deployment
kubectl get rs
kubectl get pods
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment [or] kubectl rollout undo deployment/nginx-deployment --to-revision=2
