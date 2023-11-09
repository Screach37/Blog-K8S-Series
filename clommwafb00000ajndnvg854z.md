---
title: "Navigating the DevOps Seas: A Journey from Docker to Kubernetes"
seoTitle: "Mastering DevOps: Docker to Kubernetes Deployment for React and Node"
seoDescription: "Explore the comprehensive guide to deploying ReactJS frontend and Node.js backend applications seamlessly using Docker and Kubernetes."
datePublished: Mon Nov 06 2023 08:22:16 GMT+0000 (Coordinated Universal Time)
cuid: clommwafb00000ajndnvg854z
slug: k8s-deployment-1
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/9cXMJHaViTM/upload/76710bdc5eefb462668ccbf1b5bd763a.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1699258805270/293292f1-2340-4656-b620-374e42b00d83.png
tags: docker, deployment, kubernetes, devops, nginx-ingress

---

### **Introduction:**

Embarking on a DevOps journey often feels like setting sail into uncharted waters. In our recent project, we set out to deploy a dynamic ReactJS frontend and a robust Node.js backend using Docker and Kubernetes. Join us as we unravel the tale of crafting Dockerfiles to steering through Kubernetes Deployments, Services, and Ingress resources.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1699257339834/a1f49bcd-cf11-4745-8563-6b1e9d2ccef2.png align="center")

### **Setting the Stage: Writing Multi-stage Dockerfiles**

Our journey began with the cornerstone of containerization — Docker. We adopted the best practice of multi-stage builds to optimize image sizes and enhance security. By separating the build and runtime environments, our Dockerfiles became sleek and efficient.

Dockerfile for ReactJS frontend :

```dockerfile
# Example of a Multi-stage Dockerfile for Frontend
# Stage - 1
FROM node as build  # An alias is created named "build"
WORKDIR /app

COPY package.json package-lock.json /app/
RUN npm install

COPY . /app
RUN npm run build

# Stage - 2
FROM nginx:alpine

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80
#API Call env var
ENV backendURL="http://backendconnect.com:2000"

#serving the site using Nginx
CMD ["nginx", "-g", "daemon off;"]
```

Writing the Dockerfile for Backend:

```dockerfile
# Example of a Multi-stage Dockerfile for Backend
# Stage - 1
FROM node:12 AS builder
WORKDIR /app
COPY package.json package-lock.json /app/
RUN npm install
COPY . /app
RUN npm install -g nodemon

# Stage - 2
FROM node:12-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
EXPOSE 2000
ENV backendURL="http://backendconnect.com:2000"
CMD ["nodemon", "./index.js"] 
```

### **Sailing the Kubernetes Seas: Deployments & Services**

Having containerized our applications, we set our course for Kubernetes. Deployments became our guiding stars. Configuring them allowed us to declare the desired state of our applications, ensuring scalability and availability.

```yaml
# Example of a Kubernetes Deployment for Node.js
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: your-registry/backend:latest
          ports:
            - containerPort: 2000
```

Simultaneously, Kubernetes Services acted as the lighthouses, guiding traffic to our deployed applications.

```yaml
# Example of a Kubernetes Service
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 2000
      targetPort: 2000
```

---

**Ingress: Charting External Territories with Nginx**

With our applications deployed and services exposed, we needed a way to navigate external territories. Enter Ingress, our compass in the Kubernetes realm. Using Nginx as our Ingress controller, we defined rules for external access.

```yaml
# Example of a Kubernetes Ingress with Nginx
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: your-domain.com
      http:
        paths:
          - path: /backend
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 2000
```

### **Minikube and Kubectl - Your Command-Line Companions**

Embark on local Kubernetes exploration with Minikube. Learn how to set up a local cluster for testing and development. Discover the power of kubectl commands to interact with your Kubernetes cluster, from creating deployments to checking pod logs.

```bash
# Minikube commands
minikube start --driver=vmware

# kubectl commands
kubectl apply -f frontendDeploy.yaml
kubectl apply -f backendDeploy.yaml
kubectl apply -f frontendSvc.yaml
kubectl apply -f backendSvc.yaml

kubectl get pods -o wide 
kubectl logs my-pod
```

---

**Conclusion: Unveiling the Horizon**

Our deployment journey from crafting Dockerfiles to steering Kubernetes resources has been an exhilarating expedition. With Docker as our sturdy ship and Kubernetes as our guiding star, we've navigated the complexities of modern DevOps. The horizon is vast, and the journey continues. Stay tuned for more insights, tutorials, and the upcoming blog site unveiling the full saga of our DevOps adventure.