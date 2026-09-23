# 🇪🇬 Kubernetes Ingress — Egyptian League

A hands-on Kubernetes project demonstrating how to use **Ingress** to route traffic to different Services based on the requested hostname.

## 🚀 Project Overview

This project simulates two teams:

* ⚽ **Ahly** → Nginx
* ⚽ **Zamalek** → Apache HTTP Server

Kubernetes Ingress is used as the entry point to route requests to the correct Service.

---

## 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │      Ingress        │
                    │  egyptian-league    │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
          ahly.k8s/                   zamalek.k8s/
                 │                           │
                 ▼                           ▼
        ┌────────────────┐          ┌────────────────┐
        │  Ahly Service  │          │ Zamalek Service│
        │      :80       │          │      :80       │
        └───────┬────────┘          └───────┬────────┘
                │                           │
                ▼                           ▼
          ┌───────────┐               ┌───────────┐
          │  Nginx    │               │  Apache   │
          │    Pod    │               │    Pod    │
          └───────────┘               └───────────┘
```

## 🧩 Kubernetes Components

| Component  | Name              | Image              |
| ---------- | ----------------- | ------------------ |
| Deployment | `ahly`            | `nginx`            |
| Service    | `ahly`            | ClusterIP :80      |
| Deployment | `zamalek`         | `httpd`            |
| Service    | `zamalek`         | ClusterIP :80      |
| Ingress    | `egyptian-league` | Host-based routing |

## 🌐 Ingress Rules

| Host          | Service   | Port |
| ------------- | --------- | ---: |
| `ahly.k8s`    | `ahly`    |   80 |
| `zamalek.k8s` | `zamalek` |   80 |

The Ingress uses the hostname to decide which Kubernetes Service should receive the request.

## 🔧 Local Host Configuration

Inside the Minikube environment, the hostnames are mapped to the Minikube IP:

```text
192.168.49.2 ahly.k8s
192.168.49.2 zamalek.k8s
```

This allows requests such as:

```bash
curl ahly.k8s
curl zamalek.k8s
```

to reach the Minikube Ingress.

## ✅ Testing

### Ahly

```bash
curl ahly.k8s
```

Expected result:

```text
Welcome to nginx!
```

### Zamalek

```bash
curl zamalek.k8s
```

Expected result:

```text
It works!
```

## 📌 Key Concepts Practiced

* Kubernetes Deployments
* Kubernetes Services
* ClusterIP Services
* Kubernetes Ingress
* Host-based routing
* DNS/hosts configuration
* Minikube
* Testing services with `curl`

## 🎯 What I Learned

This project demonstrates how **one Ingress can act as a single entry point** and route requests to different Kubernetes Services based on the hostname.

```text
ahly.k8s     → Ingress → ahly Service → Nginx
zamalek.k8s  → Ingress → zamalek Service → Apache
```

---

### 👨‍💻 Author

**Eslam Seadawi**

Computer Science | Backend & DevOps Enthusiast

[GitHub](https://github.com/eslams3dawi)
