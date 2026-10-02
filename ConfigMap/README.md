# Redis Configuration with ConfigMap

This example demonstrates how to use a Kubernetes **ConfigMap** to provide a custom Redis configuration file to a Redis Pod.

## Files

```text
.
├── redis-configmap.yaml
├── redis-pod.yaml
└── README.md
```

## 1. ConfigMap

The `redis-configmap.yaml` file stores the Redis configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-configmap
data:
  redis-config: |
    maxmemory 5mb
    maxmemory-policy allkeys-lru
```

The ConfigMap contains one key:

```text
redis-config
```

Its value is the Redis configuration.

---

## 2. Pod

The `redis-pod.yaml` file creates a Redis Pod and mounts the ConfigMap as a configuration file.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis:latest
      ports:
        - containerPort: 6379

      command:
        - redis-server
        - /usr/local/etc/redis/redis.conf

      volumeMounts:
        - name: redis-config-volume
          mountPath: /usr/local/etc/redis/redis.conf
          subPath: redis-config

  volumes:
    - name: redis-config-volume
      configMap:
        name: redis-configmap
        items:
          - key: redis-config
            path: redis-config
```

## 3. How the Two Files Are Connected

The relationship between the ConfigMap and the Pod is:

```text
redis-configmap.yaml
        |
        | metadata.name
        v
redis-configmap
        |
        | configMap.name
        v
redis-config-volume
        |
        | volumeMounts.name
        v
volumeMount
        |
        | subPath
        v
redis-config
        |
        | mounted as
        v
/usr/local/etc/redis/redis.conf
        |
        | command
        v
redis-server /usr/local/etc/redis/redis.conf
```

### Important Connections

### ConfigMap Name

The Pod references:

```yaml
configMap:
  name: redis-configmap
```

This must match:

```yaml
metadata:
  name: redis-configmap
```

### ConfigMap Key

The ConfigMap contains:

```yaml
data:
  redis-config:
```

The Pod selects this key:

```yaml
items:
  - key: redis-config
```

### Volume Name

The volume is named:

```yaml
name: redis-config-volume
```

The container references the same volume:

```yaml
volumeMounts:
  - name: redis-config-volume
```

### File Mount

The selected ConfigMap content is mounted at:

```text
/usr/local/etc/redis/redis.conf
```

The `subPath` selects the `redis-config` file from the volume.

### Redis Command

Redis is explicitly started with:

```yaml
command:
  - redis-server
  - /usr/local/etc/redis/redis.conf
```

This tells Redis to use the configuration file provided by the ConfigMap.

---

## 4. Deploy

Apply the ConfigMap first:

```powershell
kubectl apply -f .\redis-configmap.yaml
```

Then create the Pod:

```powershell
kubectl apply -f .\redis-pod.yaml
```

Check the Pod:

```powershell
kubectl get pods
```

Expected:

```text
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          ...
```

---

## 5. Test Redis Configuration

Open the Redis CLI:

```powershell
kubectl exec -it redis -- redis-cli
```

Check `maxmemory`:

```redis
CONFIG GET maxmemory
```

Expected:

```text
1) "maxmemory"
2) "5242880"
```

`5242880` bytes = `5 MB`.

Check the eviction policy:

```redis
CONFIG GET maxmemory-policy
```

Expected:

```text
1) "maxmemory-policy"
2) "allkeys-lru"
```

Exit:

```redis
exit
```

---

## 6. Important Note

Redis commands such as:

```redis
CONFIG GET maxmemory
```

must be executed **inside `redis-cli`**, not directly in PowerShell.

Correct:

```powershell
kubectl exec -it redis -- redis-cli
```

Then:

```redis
CONFIG GET maxmemory
```

Incorrect:

```powershell
PS C:\Users\dell> config get maxmemory
```

because `CONFIG` is a Redis CLI command, not a PowerShell command.

---

## Summary

```text
ConfigMap
    ↓
Stores Redis configuration
    ↓
Volume
    ↓
Mounts configuration inside the container
    ↓
redis.conf
    ↓
redis-server reads the file
    ↓
Redis runs with custom settings
```
