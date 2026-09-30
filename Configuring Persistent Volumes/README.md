# Kubernetes PersistentVolume (PV) & PersistentVolumeClaim (PVC)

This practice demonstrates how to create and connect a **PersistentVolume (PV)** with a **PersistentVolumeClaim (PVC)** in Kubernetes.

---

## 📁 Files

```text
Configuring Persistent Volumes/
│
├── pv.yml
├── pvc.yml
└── README.md
```

---

## 1. PersistentVolume (PV)

A **PersistentVolume** represents actual storage available inside the Kubernetes cluster.

### `pv.yml`

```yaml
apiVersion: v1

kind: PersistentVolume

metadata:
  name: pv-volume

spec:
  # Amount of storage provided by this PV
  capacity:
    storage: 2Gi

  # The volume can be mounted as Read/Write by one node
  accessModes:
    - ReadWriteOnce

  # Use a directory on the Minikube node
  hostPath:
    path: "/tmp/data"
```

### PV structure

```text
PV
│
├── Capacity: 2Gi
├── Access Mode: ReadWriteOnce
└── Storage Location: /tmp/data
```

---

## 2. PersistentVolumeClaim (PVC)

A **PVC** is a request for storage.

The PVC below requests **1Gi** of storage with `ReadWriteOnce`.

### `pvc.yml`

```yaml
apiVersion: v1

kind: PersistentVolumeClaim

metadata:
  name: pvc-storage

spec:
  # The PVC requests a volume that can be
  # mounted as Read/Write by one node
  accessModes:
    - ReadWriteOnce

  # Amount of storage requested
  resources:
    requests:
      storage: 1Gi
```

### PVC structure

```text
PVC
│
├── Request: 1Gi
└── Access Mode: ReadWriteOnce
```

---

## 3. PV ↔ PVC Binding

Kubernetes looks for a PV that satisfies the PVC requirements.

In this example:

```text
PVC Request
    │
    │ 1Gi + RWO
    ▼
┌──────────────────┐
│      PV          │
│                  │
│ Capacity: 2Gi    │
│ Access: RWO      │
└────────┬─────────┘
         │
         │ Binding
         ▼
      PVC
      1Gi
```

The **2Gi PV** can satisfy the **1Gi PVC** request because:

```text
PV capacity    = 2Gi
PVC request    = 1Gi
Access mode    = RWO
```

After successful binding:

```bash
kubectl get pv
kubectl get pvc
```

The expected relationship is:

```text
PV: pv-volume
        ▲
        │ Bound
        │
PVC: pvc-storage
```

---

## 4. Access Mode

```yaml
accessModes:
  - ReadWriteOnce
```

### `ReadWriteOnce (RWO)`

The volume can be mounted as **read/write by one node**.

```text
Node
 │
 └── Pod
      │
      └── PVC → PV
```

---

## 5. HostPath

The PV uses:

```yaml
hostPath:
  path: "/tmp/data"
```

This means the storage is backed by the `/tmp/data` directory on the Kubernetes node.

```text
Kubernetes Node
│
└── /tmp/data
      │
      ▼
     PV
      │
      ▼
     PVC
      │
      ▼
     Pod
```

> With Minikube, this path refers to the Minikube node/container environment, not automatically to a normal Windows folder.

---

## 6. Useful Commands

```powershell
# Apply the PersistentVolume
kubectl apply -f .\pv.yml

# Apply the PersistentVolumeClaim
kubectl apply -f .\pvc.yml

# Check PV
kubectl get pv

# Check PVC
kubectl get pvc

# Get detailed PV information
kubectl describe pv pv-volume

# Get detailed PVC information
kubectl describe pvc pvc-storage

# Delete the PVC
kubectl delete pvc pvc-storage

# Delete the PV
kubectl delete pv pv-volume
```

---

## 7. Complete Storage Flow

```text
                    Kubernetes Cluster
                           │
                           ▼
                    ┌─────────────┐
                    │     PV      │
                    │   2Gi RWO   │
                    │ /tmp/data   │
                    └──────┬──────┘
                           │
                         Bound
                           │
                           ▼
                    ┌─────────────┐
                    │     PVC     │
                    │   1Gi RWO   │
                    │pvc-storage  │
                    └──────┬──────┘
                           │
                         Used by
                           │
                           ▼
                         Pod
```

---

## 8. PV vs PVC

| Resource | Meaning                 | Example     |
| -------- | ----------------------- | ----------- |
| PV       | Actual storage resource | 2Gi         |
| PVC      | Request for storage     | 1Gi         |
| Pod      | Uses the PVC            | Application |

### Easy way to remember

```text
PV  = Storage
PVC = Request for Storage
Pod = Consumer

Pod → PVC → PV → Storage
```

---

## 9. Important Note About Reclaim Policy

This PV does not explicitly define a reclaim policy:

```yaml
spec:
  ...
```

So Kubernetes uses the applicable default behavior for the PV/storage setup.

You can explicitly define one:

```yaml
spec:
  persistentVolumeReclaimPolicy: Retain
```

or:

```yaml
spec:
  persistentVolumeReclaimPolicy: Delete
```

### Reclaim flow

```text
PVC deleted
     │
     ├── Retain
     │     └── PV remains
     │
     └── Delete
           └── PV/storage is removed
```

---

## 10. Summary

```text
PV:
  - Provides storage
  - Capacity = 2Gi
  - Access Mode = RWO
  - Storage path = /tmp/data

PVC:
  - Requests storage
  - Requested size = 1Gi
  - Access Mode = RWO
  - Name = pvc-storage

Relationship:

Pod
 ↓
PVC
 ↓
PV
 ↓
/tmp/data
```
