# Kubernetes Secrets

This practice demonstrates different ways to create and consume Kubernetes Secrets.

## Files Relationship

```text
                         Kubernetes Secrets
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
       db-secret.yaml    server-user-secret.yaml basic-auth.yaml
              │                 │                 │
              │                 ▼                 │
              │        server-user-pod.yaml       │
              │                 │                 │
              │                 ▼                 │
              │        Environment Variable       │
              │                                   │
              └───────────────┬───────────────────┘
                              │
                              ▼
                       Secret Data
                              │
                              ▼
                    my-volume-secret.yaml
                              │
                              ▼
                     pod-volume-secret.yaml
                              │
                              ▼
                    Secret mounted as files
```

## 1. `db-secret.yaml`

Defines a basic Kubernetes `Secret`.

It demonstrates the two ways of providing Secret values:

* `data` → values are already Base64 encoded.
* `stringData` → values are written as plaintext and Kubernetes encodes them automatically.

This file is mainly used to demonstrate **how Secret data can be stored**.

---

## 2. `server-user-secret.yaml`

Creates the `server-user` Secret.

```text
server-user-secret.yaml
          │
          │ creates
          ▼
    server-user Secret
```

It contains the `username` value that will later be consumed by a Pod.

---

## 3. `server-user-pod.yaml`

This Pod **uses the Secret created by `server-user-secret.yaml`**.

The relationship is:

```text
server-user-secret.yaml
          │
          │ creates
          ▼
    server-user Secret
          │
          │ secretKeyRef
          ▼
server-user-pod.yaml
          │
          ▼
Environment Variable
SECRET_SERVER_USERNAME
```

So instead of writing the username directly inside the Pod configuration, the Pod gets it from the Secret.

---

## 4. `basic-auth.yaml`

Creates a Secret specifically for **Basic Authentication**.

```yaml
type: kubernetes.io/basic-auth
```

It contains:

```text
username
password
```

This is independent from `server-user-secret.yaml`. It demonstrates another built-in Kubernetes Secret type.

```text
basic-auth.yaml
      │
      ▼
Basic Auth Secret
      ├── username
      └── password
```

---

## 5. `my-volume-secret.yaml`

Creates the `vol-secret` Secret that will be used for the **Secret Volume** example.

```text
my-volume-secret.yaml
          │
          │ creates
          ▼
      vol-secret
```

The Secret contains keys such as:

```text
username
password
```

These keys will later become files inside the Pod.

---

## 6. `pod-volume-secret.yaml`

This Pod **uses the `vol-secret` created by `my-volume-secret.yaml`**.

Instead of injecting the Secret as environment variables, Kubernetes mounts the Secret as a volume.

```text
my-volume-secret.yaml
          │
          │ creates
          ▼
      vol-secret
          │
          │ mounted as volume
          ▼
pod-volume-secret.yaml
          │
          ▼
/etc/secret-volume/
          ├── username
          └── password
```

Each key in the Secret becomes a file inside the mounted directory.

---

# Overall Relationship

There are **two main ways demonstrated for consuming Secrets**:

### 1. Secret → Environment Variable

```text
server-user-secret.yaml
          │
          ▼
    server-user
          │
          ▼
server-user-pod.yaml
          │
          ▼
SECRET_SERVER_USERNAME
```

### 2. Secret → Volume → Files

```text
my-volume-secret.yaml
          │
          ▼
      vol-secret
          │
          ▼
pod-volume-secret.yaml
          │
          ▼
/etc/secret-volume/
    ├── username
    └── password
```

### In short

* `db-secret.yaml` → demonstrates creating a normal Secret using `data` and `stringData`.
* `server-user-secret.yaml` → creates a Secret containing a username.
* `server-user-pod.yaml` → consumes that Secret as an **environment variable**.
* `basic-auth.yaml` → demonstrates the **Basic Authentication Secret type**.
* `my-volume-secret.yaml` → creates a Secret for the volume example.
* `pod-volume-secret.yaml` → consumes that Secret as **files through a mounted volume**.

The important idea is:

> **A Secret is created independently, then a Pod references that Secret when it needs the sensitive data.**
