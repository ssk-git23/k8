# Kubernetes Installation Tutorial

## Introduction

Kubernetes is an open-source platform for automating the deployment, scaling, and management of containerized applications. It simplifies orchestration by providing features such as load balancing, service discovery, and automated rollouts/rollbacks. This tutorial covers the installation of Kubernetes on Ubuntu Linux with a focus on setting up a master and worker nodes.

---

## High-Level Installation Process

1. Prepare your system and reset any existing Kubernetes configuration.
2. Set hostnames for the master and worker nodes.
3. Initialize the Kubernetes cluster using `kubeadm`.
4. Configure `kubectl` for cluster management.
5. Install a network plugin (e.g., Calico).
6. Join worker nodes to the master node.

---

## Prerequisites for Installation on Ubuntu Linux

1. **Operating System:** Ubuntu 20.04 or 22.04.
2. **Processor and Memory:** At least 2 CPUs and 4GB of RAM per node.
3. **Network Configuration:** Ensure the nodes can communicate with each other.
4. **Swap:** Disable swap:
   ```bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^/#/' /etc/fstab
   ```
5. **Container Runtime:** Install a supported container runtime like Docker or containerd.
6. **Kubernetes Tools:** Install `kubeadm`, `kubelet`, and `kubectl`:
   ```bash
   sudo apt update
   sudo apt install -y apt-transport-https ca-certificates curl
   curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo apt update
   sudo apt install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

---

## Key Kubernetes Tools: Purpose and Use Cases

| Component | Purpose | Use Case |
|-----------|---------|----------|
| **kubeadm** | Simplifies the process of setting up a Kubernetes cluster. It handles the cluster initialization, component setup, and configuration. | Used for setting up a master node or joining worker nodes to the cluster. |
| **kubectl** | CLI tool for managing Kubernetes clusters. It allows you to deploy applications, inspect resources, and manage cluster components. | Used by administrators and developers to interact with and manage the cluster. |
| **kubelet** | Agent that runs on each node and ensures that containers are running in a Pod as expected. It communicates with the API server. | Essential for node functionality, ensuring that containers run and stay healthy. |

---

## Step-by-Step Installation

### Step 1: Reset Existing Configuration

If you are performing a fresh installation, reset any existing Kubernetes configuration:
```bash
sudo kubeadm reset
```

### Step 2: Set Hostnames

Assign hostnames to your master and worker nodes:
```bash
sudo hostnamectl set-hostname master.example.com
sudo hostnamectl set-hostname worker-node-1.example.com
sudo hostnamectl set-hostname worker-node-2.example.com
```

### Step 3: Initialize the Kubernetes Cluster

On the master node, initialize the Kubernetes cluster:
```bash
sudo kubeadm init --ignore-preflight-errors=all
```

### Step 4: Configure kubectl

Set up `kubectl` for the current user to manage the cluster:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 5: Install Calico Networking Plugin

Apply the Calico network plugin manifest:
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
Verify the nodes:
```bash
kubectl get nodes
```

### Step 6: Join Worker Nodes

Generate the join command on the master node:
```bash
sudo kubeadm token create --print-join-command
```
Example output:
```bash
sudo kubeadm join 172.31.33.66:6443 --token qg5kgy.o1ov92iu7d50dkye --discovery-token-ca-cert-hash sha256:e3f0feef4ad831253c3535f72e17c3bddc0c631e789c621f7a130e7e798aa313
```
Run the join command on each worker node to connect them to the cluster.

---

## Verification

1. Verify the cluster nodes:
   ```bash
   kubectl get nodes
   ```
2. Check the status of the pods:
   ```bash
   kubectl get pods -A
   ```

## Next Steps

You can now use this environment to learn Kuberenetes further.

