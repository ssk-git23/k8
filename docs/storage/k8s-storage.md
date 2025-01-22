# **Kubernetes Storage: Practical Guide to concepts and examples**

---

## **Table of Contents**
1. [Overview of Storage in Kubernetes](#1-overview-of-storage-in-kubernetes)  
2. [Volumes](#2-volumes)  
   2.1 [emptyDir](#21-emptydir)  
   2.2 [hostPath](#22-hostpath)  
   2.3 [NFS Volumes](#23-nfs-volumes)  
   2.4 [Other Volume Types](#24-other-volume-types)  
3. [Ephemeral Volumes](#3-ephemeral-volumes)  
4. [Persistent Volumes (PVs)](#4-persistent-volumes-pvs)  
   4.1 [Access Modes](#41-access-modes)  
   4.2 [Persistent Volume Reclaim Policies](#42-persistent-volume-reclaim-policies)  
5. [Volume Snapshots](#5-volume-snapshots)  
6. [Storage Classes](#6-storage-classes)  
7. [Dynamic Volume Provisioning](#7-dynamic-volume-provisioning)  
8. [Storage Capacity](#8-storage-capacity)  
9. [Node-Specific Volume Limits](#9-node-specific-volume-limits)  
10. [Detailed YAML Examples](#10-detailed-yaml-examples)

---

## **1. Overview of Storage in Kubernetes**

Kubernetes storage enables applications to persist data, share files between containers, and dynamically scale data needs across distributed environments. Kubernetes abstracts underlying storage infrastructure, allowing seamless integration across different storage systems like cloud services, local disks, and network file systems.

### **Key Characteristics of Kubernetes Storage**
- **Ephemeral and Persistent Storage**: Kubernetes supports both temporary storage (like `emptyDir`) and long-term persistent storage (like Persistent Volumes).
- **Decoupling of Storage from Compute**: Storage is independent of the Pod lifecycle, enabling resilience and scalability.
- **Dynamic Provisioning**: Storage resources are provisioned on-demand using Storage Classes.
- **Support for Multiple Backends**: Kubernetes supports storage backends, including AWS EBS, GCE PD, NFS, Ceph, and more.

### **Use Cases**
1. Persisting data for stateful applications such as databases.
2. Sharing files between containers in the same Pod.
3. Enabling disaster recovery using snapshots and replication.

---

## **2. Volumes**

Kubernetes Volumes provide a way to mount storage to containers running in a Pod. Each volume exists as long as the Pod is running and provides shared access to data across containers within the Pod.

### **2.1 emptyDir**

`emptyDir` is a temporary directory that is created when a Pod is scheduled and deleted when the Pod stops.

**Use Case**: Caching temporary data during processing.  
**Example YAML**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-example
spec:
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - mountPath: "/data"
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

---

### **2.2 hostPath**

`hostPath` allows a Pod to access a file or directory on the host node.

**Use Case**: Access logs or configuration files stored on the host.  
**Example YAML**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - mountPath: "/host"
      name: host-volume
  volumes:
  - name: host-volume
    hostPath:
      path: /var/log
```

---

### **2.3 NFS Volumes**

Network File System (NFS) volumes allow multiple Pods to access the same shared storage.

**Use Case**: Shared storage for distributed applications.  
**Example YAML**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-example
spec:
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - mountPath: "/data"
      name: nfs-volume
  volumes:
  - name: nfs-volume
    nfs:
      server: 192.168.1.100
      path: /shared
```

---

### **2.4 Other Volume Types**
- **configMap**: Inject configuration data.
- **secret**: Store sensitive data securely.
- **csi**: Integrate custom storage backends.

---

## **3. Ephemeral Volumes**

Ephemeral volumes are tied to the Pod's lifecycle and are designed for temporary storage. Examples include `emptyDir`, `configMap`, and `secret`.

### **Benefits**
1. **Lightweight and Fast**: Ideal for temporary storage needs.
2. **No External Dependencies**: No need for external storage provisioning.

### **Use Case**
Temporary scratch space for a web server.

---

## **4. Persistent Volumes (PVs)**

Persistent Volumes are cluster-wide resources that provide long-term storage independent of the Pod lifecycle.

### **4.1 Access Modes**
- **ReadWriteOnce (RWO)**: Mounted by a single node for read-write.
- **ReadOnlyMany (ROX)**: Mounted by multiple nodes in read-only mode.
- **ReadWriteMany (RWX)**: Mounted by multiple nodes for read-write.

---

### **4.2 Persistent Volume Reclaim Policies**
1. **Retain**: Keeps data after release.
2. **Recycle**: Clears data and makes PV available again.
3. **Delete**: Automatically deletes the PV.

**Example YAML**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

---

## **5. Volume Snapshots**

Volume snapshots capture the state of a volume at a specific time. Snapshots enable backup and recovery operations.

**Example YAML**:
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: snapshot-example
spec:
  volumeSnapshotClassName: csi-snapclass
  source:
    persistentVolumeClaimName: pvc-example
```

---

## **6. Storage Classes**

Storage Classes enable dynamic provisioning of PVs with specific storage properties like performance and cost.

**Example YAML**:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
```

---

## **7. Dynamic Volume Provisioning**

Dynamic provisioning automatically creates PVs when a PVC is submitted.

**Example YAML**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

---

## **8. Storage Capacity**

Kubernetes monitors storage capacity for dynamic provisioning. This feature is useful for storage planning and avoiding overprovisioning.

---

## **9. Node-Specific Volume Limits**

Kubernetes limits the number of volumes attached to a Node based on its capacity.

---

## **10. Detailed YAML Examples**

All examples listed above provide a practical foundation for implementing Kubernetes storage. Each example highlights the benefits, including improved resilience, scalability, and operational efficiency.

---

