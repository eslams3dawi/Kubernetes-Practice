# Kubernetes ConfigMap Demo

This demo shows how to create and inspect Kubernetes **ConfigMaps** using:

* Configuration files
* Multiple files from a directory
* Literal key-value pairs

## 1. Create ConfigMap from a File

```powershell
kubectl create configmap config-file --from-file=C:\Path\To\db.properties
```

Creates a ConfigMap where the **filename becomes the key**.

```text
db.properties → ConfigMap key
file content  → ConfigMap value
```

### Check the ConfigMap

```powershell
kubectl get configmap
kubectl describe configmap config-file
kubectl get configmap config-file -o yaml
```

## 2. Create ConfigMap from Multiple Files

```powershell
kubectl create configmap all-keys --from-file=C:\Path\To\Config-Map-Demo
```

Every file in the directory becomes a separate key.

Example:

```text
db.properties     → key
mydbscretes.conf  → key
```

Check it:

```powershell
kubectl get configmap all-keys
kubectl get configmap all-keys -o yaml
```

## 3. Create ConfigMap from Literal Values

```powershell
kubectl create configmap newconfig-file `
  --from-literal=env=dev `
  --from-literal=ip=192.168.1.2
```

Result:

```yaml
data:
  env: dev
  ip: 192.168.1.2
```

Check it:

```powershell
kubectl get configmap newconfig-file -o yaml
```

## 4. Delete a ConfigMap

```powershell
kubectl delete configmap config-file
```

## Important Note

ConfigMap source files can have **any file extension**.

Examples:

```text
.properties
.conf
.txt
.json
.yaml
.xyz
```

When using `--from-file`:

```text
Filename      → ConfigMap Key
File Content  → ConfigMap Value
```
