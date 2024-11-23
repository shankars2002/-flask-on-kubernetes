# Deploy a Flask App on Kubernetes Using Minikube

This guide walks you through deploying a simple Flask application on Kubernetes using Minikube. It covers everything from setting up your environment to accessing your application.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Step 1: Create a Flask Application](#step-1-create-a-flask-application)
3. [Step 2: Create a Docker Image](#step-2-create-a-docker-image)
4. [Step 3: Start Minikube](#step-3-start-minikube)
5. [Step 4: Create Kubernetes Manifests](#step-4-create-kubernetes-manifests)
6. [Step 5: Access the Flask Application](#step-5-access-the-flask-application)
7. [Step 6: Clean Up](#step-6-clean-up)
8. [Explanation of Ports](#explanation-of-ports)
9. [Project Structure](#project-structure)

## Prerequisites

Before starting, ensure you have the following installed:

- **Minikube**
- **kubectl**
- **Docker**

## Step 1: Create a Flask Application

### **Create a new directory for your project:**

```bash
mkdir flask-k8s
cd flask-k8s


### **Create a new Flask application in `app.py`:**

```bash
touch app.py
```

### Add the following code to `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Flask on Kubernetes!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### **Create a `requirements.txt` file:**

```bash
touch requirements.txt
```

### Add the following content to `requirements.txt`:

```text
Flask==2.0.3
```

## Step 2: Create a Docker Image

### **Create a `Dockerfile` to containerize your Flask application:**

```bash
touch Dockerfile
```

### Add the following content to the `Dockerfile`:

```dockerfile
FROM python:3.8-slim

WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .

CMD ["python", "app.py"]
```

### **Build the Docker image:**

```bash
docker build -t flask-app .
```

### **Verify the image is built:**

```bash
docker images
```

## Step 3: Start Minikube

### **Start Minikube:**

```bash
minikube start
```

### **Use Minikube's Docker daemon:**

```bash
eval $(minikube docker-env)
```

### **Rebuild the Docker image inside Minikube:**

```bash
docker build -t flask-app .
```

## Step 4: Create Kubernetes Manifests

### **Create a `flask.yaml` file with the following content:**

```bash
touch flask.yaml
```

### Add the following content to `flask.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: flask-app
        ports:
        - containerPort: 5000

---
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: NodePort
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
```

### **Deploy the app:**

```bash
kubectl apply -f flask.yaml
```

### **Check pod status:**

```bash
kubectl get pods
```

### **Check service status:**

```bash
kubectl get svc
```

## Step 5: Access the Flask Application

### **Find the Minikube IP:**

```bash
minikube ip
```

### **Get the external port assigned to `flask-service`:**

```bash
kubectl get svc flask-service
```

Look for the `NodePort`, e.g., `32150`.

### **Access the app in your browser:**

```text
http://<minikube-ip>:<NodePort>
```

Example:

```text
http://192.168.49.2:32150
```

## Step 6: Clean Up

### **To delete the resources and stop Minikube:**

```bash
kubectl delete -f flask.yaml
minikube stop
```

## Explanation of Ports

| Port Type     | Description                                                |
|---------------|------------------------------------------------------------|
| **ContainerPort** | The port the Flask app listens on inside the pod.     |
| **TargetPort**    | The port Kubernetes forwards traffic to inside the pod. |
| **NodePort**      | The external port Minikube assigns to expose the service. |

## Project Structure


```text
flask-k8s/
├── app.py
├── requirements.txt
├── Dockerfile
├── flask.yaml
├── README.md
```

