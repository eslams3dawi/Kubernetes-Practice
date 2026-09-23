# Kubernetes Multi-App Ingress Lab

A hands-on Kubernetes setup using **Minikube**, with multiple Deployments, Services, and one Ingress routing traffic by hostname and path.

## Architecture

```text
                    seadawi.com
                         │
                  ┌──────▼──────┐
                  │    Ingress   │
                  │ multiple-    │
                  │   ingress    │
                  └──────┬──────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       `/`           `/app2`         `/app3`
          │              │              │
       app1 Service   app2 Service   app3 Service
          │              │              │
       6 Pods          7 Pods          8 Pods
```

## 1. Create Deployments

```bash
# Create Deployments
kubectl create deployment app1 --image=httpd:latest
kubectl create deployment app2 --image=gcr.io/google-samples/hello-app:1.0
kubectl create deployment app3 --image=gcr.io/google-samples/hello-app:2.0

# Scale Deployments
kubectl scale deployment app1 --replicas=6
kubectl scale deployment app2 --replicas=7
kubectl scale deployment app3 --replicas=8

# Check Deployments
kubectl get deployments
```

## 2. Create Services

```bash
# Expose app1 on port 80
kubectl expose deployment app1 --name=app1 --type=NodePort --port=80

# Expose app2 on port 8080
kubectl expose deployment app2 --name=app2 --type=NodePort --port=8080

# Expose app3 on port 8080
kubectl expose deployment app3 --name=app3 --type=NodePort --port=8080

# Check Services
kubectl get services
```

## 3. Enable NGINX Ingress

```bash
# Enable Ingress controller
minikube addons enable ingress

# Check Ingress controller
kubectl get pods -n ingress-nginx
```

## 4. Ingress Configuration

Create `multiple-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multiple-ingress
spec:
  rules:
  - host: seadawi.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: app1
              port:
                number: 80

        - path: /app2
          pathType: Prefix
          backend:
            service:
              name: app2
              port:
                number: 8080

        - path: /app3
          pathType: Prefix
          backend:
            service:
              name: app3
              port:
                number: 8080
```

Apply it:

```bash
kubectl apply -f multiple-ingress.yaml

# Check Ingress
kubectl get ingress

# Get detailed information
kubectl describe ingress multiple-ingress
```

## 5. Host Configuration

Get the Minikube IP:

```bash
minikube ip
# 192.168.49.2
```

Inside Minikube:

```bash
minikube ssh

# Map the domain to Minikube
sudo /bin/sh -c 'echo "192.168.49.2 seadawi.com" >> /etc/hosts'
```

## 6. Test Routing

```bash
# Root path → app1
curl seadawi.com/

# /app2 → app2
curl seadawi.com/app2

# /app3 → app3
curl seadawi.com/app3
```

Expected routing:

```text
seadawi.com/        → app1
seadawi.com/app2    → app2
seadawi.com/app3    → app3
```

## 7. Useful Checks

```bash
# View all resources
kubectl get all

# View Ingresses
kubectl get ingress

# Describe Ingress
kubectl describe ingress multiple-ingress

# Open Kubernetes Dashboard
minikube dashboard

# Access Minikube VM
minikube ssh
```

## Current Setup

```text
Deployments:
app1 → 6 replicas
app2 → 7 replicas
app3 → 8 replicas

Services:
app1 → NodePort → 80
app2 → NodePort → 8080
app3 → NodePort → 8080

Ingress:
multiple-ingress
Host: seadawi.com
Controller: nginx
```

## Key Concepts

- **Deployment** manages and maintains Pod replicas.
- **Service** provides stable networking to Pods.
- **Ingress** routes HTTP/HTTPS traffic to Services.
- **Path-based routing** sends different URL paths to different applications.
- **Minikube** provides the local Kubernetes environment.
