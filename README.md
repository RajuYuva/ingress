# Exposing a Sample Python Application with Kubernetes Ingress on Minikube

This guide walks you through deploying a sample Python application using Kubernetes resources (Deployment, Service, and Ingress) and exposing it via the Nginx Ingress Controller on Minikube. The sample application uses the image `abhishekf5/python-sample-app-demo:v1`.

## Prerequisites

- **Minikube** installed and running.
- **kubectl** command-line tool installed.
- Basic knowledge of Kubernetes concepts such as Deployments, Services, and Ingress.
- Access to the [Kubernetes Ingress documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/) and the [Minikube Ingress tutorial](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/).

## Setup Steps

### 1. Start Minikube and Enable Ingress

First, start Minikube (if not already running):

```bash
minikube start
```

Enable the Nginx Ingress Controller on Minikube:

```bash
minikube addons enable ingress
```

*Note: This command sets up the Nginx Ingress Controller in the `ingress-nginx` namespace, allowing it to watch for Ingress resources.*

### 2. Create Kubernetes Resource Files

Create the following YAML files in your working directory:

#### **deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-python
  labels:
    app: sample-python
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-python
  template:
    metadata:
      labels:
        app: sample-python
    spec:
      containers:
      - name: python-app
        image: abhishekf5/python-sample-app-demo:v1
        ports:
        - containerPort: 8000
```

This Deployment creates two replicas of the sample Python application.

#### **service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-app
spec:
  type: NodePort
  selector:
    app: sample-python
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

The Service exposes the application on port 80 and maps it to container port 8000. A fixed `nodePort` (30007) is also defined.

#### **ingress.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: /demo
        backend:
          service:
            name: python-app
            port:
              number: 80
```

The Ingress resource routes HTTP requests with the host `foo.bar.com` and path `/demo` to the `python-app` service.

### 3. Deploy the Application

Apply the YAML files in the following order:

1. **Deploy the sample application**

   ```bash
   kubectl apply -f deployment.yaml
   ```

2. **Create the Service**

   ```bash
   kubectl apply -f service.yaml
   ```

3. **Create the Ingress resource**

   ```bash
   kubectl apply -f ingress.yaml
   ```

Verify that the resources have been created:

```bash
kubectl get pods          # Check the running pods for the application.
kubectl get svc           # Confirm the service is running.
kubectl get ingress       # Ensure the ingress resource is in place.
kubectl get pods -n ingress-nginx   # Check that the Nginx Ingress Controller pods are running.
```

### 4. Testing the Deployment

#### Accessing the Application via Ingress

1. **Determine Minikube IP:**

   Get the Minikube IP address (usually something like `192.168.49.2`):

   ```bash
   minikube ip
   ```

2. **Test using `curl`:**

   Use `curl` from within the Minikube VM or your local terminal:

   ```bash
   curl -L http://192.168.49.2:30007/demo
   ```

   *Note: The path `/demo` is specified in the Ingress rule. If you are testing via hostname resolution (using `foo.bar.com`), make sure to add an entry to your hosts file mapping `foo.bar.com` to the Minikube IP.*

3. **Alternatively, Get the Service URL:**

   Minikube provides a convenient command to fetch the service URL:

   ```bash
   minikube service python-app --url
   ```

   This command returns the URL where the service is accessible.

### 5. Configuring Hostname (Optional)

If you prefer accessing the application with the hostname specified in the Ingress (`foo.bar.com`):

- Edit your local `/etc/hosts` (or equivalent) file and add:

  ```
  192.168.49.2  foo.bar.com
  ```

- Then test the Ingress URL:

  ```bash
  curl -L http://foo.bar.com/demo
  ```

## Conclusion

By following these steps, you have:

- Deployed a Python application using Kubernetes Deployment.
- Exposed it via a NodePort Service.
- Configured an Ingress resource to route external traffic to the service.
- Deployed and used the Nginx Ingress Controller on Minikube.

For more details, refer to the [Kubernetes Ingress documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/) and the [Minikube Ingress tutorial](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/).

Happy Deploying!

---
