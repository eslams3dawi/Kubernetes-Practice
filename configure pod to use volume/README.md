# Redis Pod with EmptyDir Volume

## Overview

This practice demonstrates how to configure a Kubernetes Pod with a volume and mount that volume inside a Redis container.

## Kubernetes Configuration

The Pod uses an `emptyDir` volume named `redis-storage`.

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: redis
  labels:
    app: redis

spec:
  containers:
    - name: redis
      image: redis:latest

      # Mount the volume inside the container
      volumeMounts:
        - name: redis-storage
          mountPath: /data/redis

  # Define the volume
  volumes:
    - name: redis-storage
      emptyDir: {}
```

## Important Concepts

### Volume

A **Volume** defines the storage available to the Pod.

```yaml
volumes:
  - name: redis-storage
    emptyDir: {}
```

### Volume Mount

A **Volume Mount** specifies where the volume appears inside the container.

```yaml
volumeMounts:
  - name: redis-storage
    mountPath: /data/redis
```

So:

```text
redis-storage
      │
      │ mount
      ▼
/data/redis
```

## emptyDir

`emptyDir` creates temporary storage for the Pod.

* The storage is created when the Pod is assigned to a node.
* Containers inside the same Pod can share it.
* The data remains available while the Pod exists.
* When the Pod is deleted, the `emptyDir` data is deleted.

## Apply the Configuration

```bash
kubectl apply -f redis.yml
```

Check the Pod:

```bash
kubectl get pods
```

Expected:

```text
NAME    READY   STATUS
redis   1/1     Running
```

## Access the Container

```bash
kubectl exec -it redis -- /bin/bash
```

Check the mounted directory:

```bash
cd /data/redis
ls
```

Create a test file:

```bash
echo "Hello world" >> file.txt
```

Read the file:

```bash
cat file.txt
```

Expected:

```text
Hello world
```

## Verify the Volume

```bash
kubectl describe pod redis
```

The output should show:

```text
Mounts:
  /data/redis from redis-storage (rw)

Volumes:
  redis-storage:
    Type: EmptyDir
```

## Key Takeaway

```text
Volume
  ↓
redis-storage
  ↓
volumeMount
  ↓
/data/redis
  ↓
Redis Container
```

**Volume = where the storage is defined.**

**VolumeMount = where that storage is mounted inside the container.**
