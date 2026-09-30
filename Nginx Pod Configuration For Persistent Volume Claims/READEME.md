# Nginx Pod Configuration For Persistent Volume Claims

## Overview

This practice demonstrates how to use a **PersistentVolumeClaim (PVC)** with an Nginx Pod.

```text
Pod
 ↓
VolumeMount
 ↓
PVC
 ↓
PV
 ↓
Minikube Storage
```

## Configuration

The Pod uses:

```yaml
volumeMounts:
  - name: storage-volume
    mountPath: /usr/share/nginx/html

volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: pvc-storage
```

## Commands

```bash
# Create PVC and Pod
kubectl apply -f pod-volume.yml

# Check PVC
kubectl get pvc

# Check PV
kubectl get pv

# Check Pod
kubectl get pods

# Describe Pod
kubectl describe pod web-server
```

## Test Storage

```bash
# Create a file inside the mounted volume
kubectl exec web-server -- touch /usr/share/nginx/html/seadawi

# Access Minikube
minikube ssh

# Check the storage directory
ls -l /tmp/hostpath-provisioner/default/pvc-storage
```

Expected:

```text
seadawi
```

## Key Points

* PVC requested **5Gi** of storage.
* Kubernetes dynamically created a **5Gi PV**.
* The PVC became **Bound** to the PV.
* Nginx mounts the PVC at `/usr/share/nginx/html`.
* The file created in the Pod appears in the Minikube storage.
* The PV uses the `Delete` reclaim policy.
