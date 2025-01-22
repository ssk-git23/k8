# 2. Mounting Pod Files to Host with hostPath

## 2.1 Overview
Kubernetes allows Pods to access files and directories from the host node through a volume type called `hostPath`. This type of volume mounts files or directories from the node’s filesystem into the Pod’s filesystem, enabling direct interaction with the underlying host’s files. This is useful when you need to read/write files or mount configuration files directly from the host.

## 2.2 Concept
A `hostPath` volume is defined within the Pod’s YAML configuration, pointing to a specific directory or file on the node’s filesystem. When you specify `hostPath`, Kubernetes mounts that path directly into the container’s filesystem, making the host file accessible for operations such as logging, configuration, or even persistent storage.

## 2.3 Benefits
- **Access to Host Files**: Provides direct access to files and directories on the host, useful for logging, configuration, or debugging.
- **No Need for Network Setup**: Eliminates the need for a shared network filesystem.
- **Simple and Quick**: Mounting host files is straightforward and doesn’t require additional setup beyond defining the `hostPath`.

## 2.4 Use Cases
- **Logging**: Mount the `/var/log` directory of the node to access logs stored on the host.
- **Configuration**: Load configuration files directly from the host to ensure the application inside the container uses the latest configurations.
- **Persistent Storage**: If you have data stored locally on the host node, you can mount it into the Pod for persistent storage.

## 2.5 Real-World Scenario
Consider a Kubernetes environment where an application requires access to logs or configuration files on the host. Using `hostPath`, you can mount the `/etc/config` directory of the host into the container, ensuring that the container always uses the latest configuration changes made on the host node.

## 2.6 Implementation Example

Here’s an example YAML configuration where a Pod uses `hostPath` to mount a directory from the host node to the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
  - name: myapp-container
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: hostpath-volume
  volumes:
  - name: hostpath-volume
    hostPath:
      path: /tmp/host-directory
      type: Directory
```

In this example:
- We mount the `/tmp/host-directory` from the host to the `/usr/share/nginx/html` directory inside the container.
- Any file modifications in `/tmp/host-directory` on the host will reflect inside the container at `/usr/share/nginx/html`.

## 2.7 Verification Steps

1. **Create the Directory on the Host**:
   Ensure that the directory `/tmp/host-directory` exists on both the worker nodes (not on the master node):
   ```bash
   mkdir /tmp/host-directory
   ```

2. **Deploy the Pod**:
   Apply the YAML configuration to your Kubernetes cluster:
   ```bash
   kubectl apply -f hostpath-example.yaml
   ```

3. **Check Pod Status**:
   Verify that the Pod is running:
   ```bash
   kubectl get pods
   ```

4. **Check Mounted Directory**:
   Enter the container and verify that the `hostPath` volume is mounted correctly:
   ```bash
   kubectl exec -it hostpath-example -- /bin/bash
   ls /usr/share/nginx/html
   ```

You should be able to see the files from the `/tmp/host-directory` of the host mounted in the container's `/usr/share/nginx/html` directory.
