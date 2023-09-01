# Setup CI Server

<img src="https://img.shields.io/badge/Parag%20Pallav%20Singh-black?style=for-the-badge&logo=docker"/>
<img src="https://img.shields.io/badge/MiniCube Reddit App-002a5c?style=for-the-badge&logo=reddit"/>

**Launch an ec2 instance with Ubuntu- t2.micro AMI type**
```
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```
<kbd><img width="629" alt="image" src="https://github.com/paragpallavsingh/reddit-clone-k8s-ingress/assets/40052830/d07f8b9d-c624-48e9-8883-36cd1a8aef36">


## Clone the project
```
mkdir projects
cd projects/
git clone https://github.com/paragpallavsingh/reddit-clone-k8s-ingress.git
cat Dockerfile
docker build . -t paragpallavsingh/reddit-clone-image:latest
sudo docker images
```

## Push to Dockerhub
```
docker login
docker push paragpallavsingh/reddit-clone-image:latest
```

# Setup Deployment
```
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```
<kbd>![image](https://github.com/paragpallavsingh/reddit-clone-k8s-ingress/assets/40052830/c2829e3d-7a0b-4601-b112-e2f937f45561)

## Installation of Minikube & Kubectl
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo snap install kubectl --classic
minikube start --driver=docker
```
<kbd>![image](https://github.com/paragpallavsingh/reddit-clone-k8s-ingress/assets/40052830/03793181-aa39-4b8a-9ecf-00be3331df47)


## Create manifest files in a project folder
```
mkdir -p project/reddit-app
cd project/reddit-app
kubectl create namespace reddit-ns
```
<kbd><img width="326" alt="image" src="https://github.com/paragpallavsingh/reddit-clone-k8s-ingress/assets/40052830/12bbc0e8-f30a-4458-b805-e2bebda1bbee">

### deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: rohanrustagi18/redditclone
        ports:
        - containerPort: 3000
```
Run `kubectl apply -f deployment.yml -n reddit-ns` after saving the above file

### service.yml
```
apiVersion: v1
# Indicates this as a service
kind: Service
metadata:
  # Service name
  name: reddit-clone-service
spec:
  selector:
    # Selector for Pods
    app: reddit-clone
  ports:
    # Port Map
  - port: 3000
    targetPort: 3000
    protocol: TCP
  type: LoadBalancer
```
Run `kubectl apply -f service.yml -n reddit-ns` after saving the above file

**To get the URL for your app:** 
```
minikube service reddit-clone-service -n reddit-ns created --url
```
**Expose the App Deployment:**
```
kubectl expose deployment reddit-clone-deployment -n reddit-ns --type=NodePort
```
**Expose the App Service:**
```
kubectl port-forward svc/reddit-clone-service -n reddit-ns 3000:3000 --address 0.0.0.0
```
<kbd><img width="960" alt="image" src="https://github.com/paragpallavsingh/reddit-clone-k8s-ingress/assets/40052830/0529e6ff-5e35-4fc4-ad29-dcd30918585d">

### ingress.yml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
```
**Enable minikube ingress:** 
```
minikube addons enable ingress
```

**Check the Current settings of addons in Minikube:**
```
minikube addons list
```

**Run ingress file:** 
```
kubectl apply -f ingress.yml -n reddit-app
```

**Check the created ingress file:** 
```
kubectl get ingress
```

Test your ingress: `curl -L domain.com/test`

<kbd><img width="818" alt="ingress" src="https://github.com/paragpallavsingh/reddit-clone-k8s-ingress/assets/40052830/9d8f1266-dbd1-42ad-8094-98436450fe61">


