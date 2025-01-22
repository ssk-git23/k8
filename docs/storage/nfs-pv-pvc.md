# 6. Configuring Pod Using NFS-Based PV and PVC

## 6.1 Overview
In Kubernetes, an NFS (Network File System) volume allows Pods to mount shared directories from an external NFS server. Using NFS provides a scalable way to share data between multiple Pods and nodes, which can be useful for scenarios requiring persistent storage that needs to be shared across multiple Pods. This tutorial explains how to configure a Persistent Volume (PV) and Persistent Volume Claim (PVC) with an NFS server and mount the volume in a Pod.

## 6.2 Concept
An NFS-based volume is a shared network file system that multiple Pods can access concurrently. It is especially useful when you need persistent data that should be accessible across Pods on different nodes. By using NFS with Kubernetes, you can ensure that your application has access to the same data, regardless of the node on which the Pod is running. Kubernetes supports NFS volumes, allowing Pods to dynamically mount shared directories for persistent storage.

### How It Works:
- **NFS Server**: The NFS server exports a directory, which is then mounted by the Pods via a Persistent Volume (PV).
- **PV**: The Persistent Volume uses the NFS server's exported directory as its source.
- **PVC**: A Persistent Volume Claim is created to request storage from the PV.
- **Pod**: The Pod is configured to use the PVC, thus mounting the NFS volume.

## 6.3 Benefits
- **Shared Storage**: NFS allows multiple Pods to share the same storage, which is ideal for applications that require access to the same data.
- **Scalability**: It is scalable across nodes and Pods, providing flexibility for growing applications.
- **Centralized Data Management**: Data is stored centrally on the NFS server, which makes it easier to manage and back up.
- **Cross-Node Access**: Pods can access the same data even if they are running on different nodes.

## 6.4 Use Cases
- **Shared Configuration Files**: When multiple Pods need access to shared configuration files or static data.
- **Database Storage**: For database applications that require a consistent, shared data storage location.
- **Log Aggregation**: Aggregating logs from multiple Pods into a shared directory accessible by other Pods for processing or analysis.

## 6.5 Real-World Scenario
In an e-commerce application, multiple Pods run various microservices like product catalog, customer management, and order processing. These microservices need access to shared configuration files and logs. By using an NFS-based Persistent Volume, all Pods can access the same directory to retrieve configuration files and write logs, ensuring data consistency and easy log aggregation.

## 6.6 Implementation Example

### 6.6.1 Step 1: Set Up an NFS Server
First, you need an NFS server running, exporting a directory. For the sake of simplicity, assume the NFS server is already set up and exporting the directory `/srv/nfs/data`.

### 6.6.2 Step 2: Create the NFS Persistent Volume
Define a Persistent Volume (PV) that uses the NFS share.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  storageClassName: manual
  nfs:
    path: /mnt/nfs_mount
    server: 172.31.29.84  # Replace with the actual NFS server IP address (our nfs server is running on master node that has this IP).
```

### Explanation of the YAML:
- **nfs.path**: Specifies the directory on the NFS server to be used as the Persistent Volume.
- **nfs.server**: The IP address of the NFS server.
- **accessModes**: `ReadWriteMany` means that multiple Pods can mount the volume simultaneously for read and write access.
- **persistentVolumeReclaimPolicy**: Retains the volume after it is released, rather than deleting it.
- **capacity**: The storage size requested from the NFS server (1Gi in this case).

### 6.6.3 Step 3: Create the Persistent Volume Claim (PVC)
Create a Persistent Volume Claim (PVC) that will request the NFS-based storage.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

### Explanation of the YAML:
- **accessModes**: Matches the PV's access mode (`ReadWriteMany`), meaning the PVC will request a volume that allows multiple Pods to read and write simultaneously.
- **resources.requests.storage**: Requests 1Gi of storage from the PV.

### 6.6.4 Step 4: Create the Pod Using the NFS PVC
Create a Pod that mounts the NFS-based volume using the PVC.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: nfs-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: nfs-volume
    persistentVolumeClaim:
      claimName: nfs-pvc
```

### Explanation of the YAML:
- **volumeMounts.mountPath**: Mounts the volume to the `/usr/share/nginx/html` directory in the container.
- **volumes.persistentVolumeClaim.claimName**: Refers to the PVC (`nfs-pvc`) requesting the NFS storage.

## 6.7 Verification Steps

1. **Create the NFS Persistent Volume**:
   Apply the PV YAML to create the Persistent Volume:
   ```bash
   kubectl apply -f nfs-pv.yaml
   ```

2. **Create the PVC**:
   Apply the PVC YAML to create the Persistent Volume Claim:
   ```bash
   kubectl apply -f nfs-pvc.yaml
   ```

3. **Create the Pod**:
   Apply the Pod YAML to create the Pod that uses the NFS PVC:
   ```bash
   kubectl apply -f nfs-pod.yaml
   ```

4. **Verify the PVC is Bound to the PV**:
   Check if the PVC is correctly bound to the PV:
   ```bash
   kubectl get pvc nfs-pvc
   ```

   The `STATUS` should be `Bound`.

5. **Check the Pod Status**:
   Check the status of the Pod to ensure it is running:
   ```bash
   kubectl get pods
   ```

6. **Verify the Mounted Volume**:
   Enter the Pod and check if the volume is correctly mounted to the specified path:
   ```bash
   kubectl exec -it nfs-pod -- /bin/bash
   ls /usr/share/nginx/html
   ```

   You should see the content from the NFS-mounted directory (`/mnt/nfs_mount`).
