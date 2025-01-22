# NFS Server Setup and Configuration in Kubernetes Lab Tutorial

In this tutorial, we will walk through the process of setting up an NFS server on a Debian-based Linux machine and configuring it to share files with Kubernetes pods across multiple nodes. We will also go through the steps to configure a Kubernetes Persistent Volume (PV), Persistent Volume Claim (PVC), and a Pod YAML to mount the NFS file system.

## Table of Contents
1. [**Setting Up NFS Server on Debian**](#setting-up-nfs-server-on-debian)
   1. [Install NFS Server](#install-nfs-server)
   2. [Configure NFS Exports](#configure-nfs-exports)
   3. [Start NFS Server](#start-nfs-server)
   4. [Verify NFS Server](#verify-nfs-server)
2. [**Configuring the Client Node**](#configuring-the-client-node)
   1. [Install NFS Client](#install-nfs-client)
   2. [Mount NFS File System](#mount-nfs-file-system)
   3. [Verify NFS Mount](#verify-nfs-mount)
3. [**Creating Kubernetes Persistent Volume (PV) Using NFS**](#creating-kubernetes-persistent-volume-pv-using-nfs)
4. [**Creating Kubernetes Persistent Volume Claim (PVC)**](#creating-kubernetes-persistent-volume-claim-pvc)
5. [**Creating a Pod to Use NFS Volume**](#creating-a-pod-to-use-nfs-volume)
6. [**Verifying the NFS Volume in Kubernetes**](#verifying-the-nfs-volume-in-kubernetes)

---

## 1. Setting Up NFS Server on Debian

In this section, we will install and configure an NFS server on a Debian machine. This NFS server will be used to share files with Kubernetes pods.

### 1.1 Install NFS Server

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

This will install the necessary packages to run the NFS server on your Debian machine.

### 1.2 Configure NFS Exports

The NFS server will share a specific directory. For this example, let's create a directory `/mnt/nfs_share` to share.

```bash
sudo mkdir -p /mnt/nfs_share && chmod -R 777 /mnt/nfs_share
```

Next, we need to configure which directories the server will export. Edit the `/etc/exports` file:

```bash
sudo nano /etc/exports
```

Add the following line to the file to export `/mnt/nfs_share` to clients:

```
/mnt/nfs_share *(rw,sync,no_subtree_check)
```

- `*` allows any client to access the directory (you can limit it to specific IPs if needed).
- `rw` allows read and write permissions.
- `sync` ensures that changes are written to disk before responding.
- `no_subtree_check` disables subtree checking for performance reasons.

### 1.3 Start NFS Server

Once the exports are configured, apply the changes and start the NFS server:

```bash
sudo exportfs -a
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

This will start the NFS server and enable it to start on boot.

### 1.4 Verify NFS Server

To verify that the NFS server is running, use the following command:

```bash
sudo systemctl status nfs-kernel-server
```

You should see that the NFS server is active and running.

To check which directories are being exported, run:

```bash
sudo exportfs -v
```

This will display the exported directories and their permissions.

---

## 2. Configuring the Client Node

In this section, we will configure a client node to mount the NFS file system so that it can access the shared files.

### 2.1 Install NFS Client

On the client node, you need to install the NFS client utilities:

```bash
sudo apt update
sudo apt install nfs-common
```

### 2.2 Mount NFS File System

Next, mount the NFS share to a local directory on the client node. For example, mount it to `/mnt/nfs_mount`:

```bash
sudo mkdir -p /mnt/nfs_mount
sudo mount <NFS_SERVER_IP>:/mnt/nfs_share /mnt/nfs_mount
```

Replace `<NFS_SERVER_IP>` with the actual IP address of your NFS server. If the server IP is `192.168.1.100`, the command will look like this:

```bash
sudo mount 192.168.1.100:/mnt/nfs_share /mnt/nfs_mount
```

### 2.3 Verify NFS Mount

You can verify that the NFS mount is successful by listing the files in the mount directory:

```bash
ls /mnt/nfs_mount
```

You should be able to see the contents of the `/mnt/nfs_share` directory from the NFS server.

---

## 3. Creating Kubernetes Persistent Volume (PV) Using NFS

Now that we have the NFS server running and accessible by the client, we will create a Kubernetes Persistent Volume (PV) that uses the NFS share.

### 3.1 Define the Persistent Volume

Create a `nfs-pv.yaml` file:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /mnt/nfs_share
    server: <NFS_SERVER_IP>
```

Replace `<NFS_SERVER_IP>` with the actual IP address of your NFS server.

### 3.2 Apply the PV Configuration

Apply the PV configuration to the Kubernetes cluster:

```bash
kubectl apply -f nfs-pv.yaml
```

### 3.3 Verify the PV

To verify that the Persistent Volume is created, run:

```bash
kubectl get pv
```

You should see the `nfs-pv` in the list of persistent volumes.

---

## 4. Creating Kubernetes Persistent Volume Claim (PVC)

Next, we will create a Persistent Volume Claim (PVC) to request the storage from the `nfs-pv`.

### 4.1 Define the PVC

Create a `nfs-pvc.yaml` file:

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

### 4.2 Apply the PVC Configuration

Apply the PVC configuration to the Kubernetes cluster:

```bash
kubectl apply -f nfs-pvc.yaml
```

### 4.3 Verify the PVC

To verify that the PVC is created, run:

```bash
kubectl get pvc
```

You should see the `nfs-pvc` in the list of persistent volume claims.

---

## 5. Creating a Pod to Use NFS Volume

Now, we will create a Pod that uses the NFS Persistent Volume Claim.

### 5.1 Define the Pod YAML

Create a `nfs-pod.yaml` file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-pod
spec:
  containers:
  - name: app
    image: busybox
    command: [ "sleep", "3600" ]
    volumeMounts:
    - mountPath: /mnt/nfs
      name: nfs-volume
  volumes:
  - name: nfs-volume
    persistentVolumeClaim:
      claimName: nfs-pvc
```

In this example:
- The `nfs-pod` will mount the NFS volume at `/mnt/nfs`.
- The `busybox` container will be used as a simple container that stays running (`sleep 3600`).

### 5.2 Apply the Pod Configuration

Apply the Pod configuration to the Kubernetes cluster:

```bash
kubectl apply -f nfs-pod.yaml
```

### 5.3 Verify the Pod

To verify that the Pod is running, use the following command:

```bash
kubectl get pods
```

You should see the `nfs-pod` in the list of pods.

---

## 6. Verifying the NFS Volume in Kubernetes

Finally, verify that the Pod has successfully mounted the NFS volume.

### 6.1 Access the Pod

To access the Pod, use:

```bash
kubectl exec -it nfs-pod -- /bin/sh
```

### 6.2 Verify the Mount

Once inside the container, check if the NFS volume is mounted:

```bash
ls /mnt/nfs
```

You should see the contents of the `/mnt/nfs_share` directory from the NFS server.

---

### Conclusion

In this tutorial, you have learned how to:
1. Set up an NFS server on a Debian-based machine.
2. Configure an NFS client to mount the file system.
3. Create a Persistent Volume (PV), Persistent Volume Claim (PVC), and Pod in Kubernetes to access the NFS volume.

This process provides a simple and effective way to share storage across multiple Kubernetes nodes using NFS.
