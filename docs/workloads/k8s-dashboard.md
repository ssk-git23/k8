# Tutorial: Setting Up Kubernetes Dashboard on an Existing Cluster

## Introduction
In this tutorial, you'll learn how to set up the **Kubernetes Dashboard**, a web-based user interface for managing Kubernetes clusters. Kubernetes Dashboard provides an intuitive way to visualize and interact with your cluster's resources, making it easier to monitor and manage your applications.

## What is the Kubernetes Dashboard?
The **Kubernetes Dashboard** is a general-purpose, web-based UI for Kubernetes clusters. It allows users to:
- View and manage applications deployed on the cluster.
- Inspect and troubleshoot cluster resources.
- Monitor resource usage and health status.
- Create and manage Kubernetes objects like Pods, Deployments, Services, and more.
- Access cluster metrics and logs.

### How Kubernetes Dashboard Helps:
- **Visibility:** Provides a graphical representation of your cluster's resources, allowing you to easily visualize the current state of your applications and infrastructure.
- **Simplicity:** Simplifies complex Kubernetes operations with an intuitive user interface, making it accessible to developers, operators, and administrators alike.
- **Efficiency:** Enables common tasks such as deploying applications, scaling resources, and troubleshooting issues directly from the dashboard.
- **Monitoring:** Allows you to monitor resource usage, health status, and application performance metrics.
- **Centralized Management:** Manage multiple Kubernetes clusters from a single dashboard, streamlining administrative tasks.

Now, let's proceed with the step-by-step setup of Kubernetes Dashboard on your existing Kubernetes cluster.


## Prerequisites
- **Helm** (package manager for Kubernetes).
- A web browser on the worker node (instructions for installing Google Chrome are provided at the end).

---

## Step 1: Install Helm on Ubuntu
To install Helm, use the following commands:

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

---

## Step 2: Deploy Kubernetes Dashboard

### 2.1 Add the Kubernetes Dashboard Repository
Add the Kubernetes Dashboard Helm repository with the following command:

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

### 2.2 Deploy the Dashboard
Deploy the Kubernetes Dashboard using Helm:

```bash
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

### 2.3 Check Deployment Status
Check the deployment status with the following command:

```bash
helm status kubernetes-dashboard --namespace kubernetes-dashboard
```

---

## Step 3: Access the Dashboard

### 3.1 Port Forward to Access the Dashboard
Run the following command to port-forward and access the Dashboard:

Note: 8444 is used since 8443 is used by Graphical server in some Debian machines.

```bash
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8444:443
```

### 3.2 Obtain Access Token
To access the Dashboard, you need to create an admin-user and set up credentials.

Apply the following YAML file:

```bash
kubectl apply -f k8s-dashboard.yaml
```

You can download the YAML file from [this GitHub repository](https://github.com/devopscert202/ckacoursenov24/blob/main/k8s/labs/k8s-dashboard.yaml).

### 3.3 Get the Token to Access the Dashboard
Generate the token with the following command:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Copy the generated token and use it on the Dashboard login page that prompts for a **Bearer Token**.

### 3.4 Access the Dashboard
Open your browser and navigate to:

```
https://localhost:8444
```

Accept the SSL certificate warning and explore various workloads like **Pods**, **Deployments**, and **Services**. You can also try scaling your deployments using the graphical interface.

---

## Installing Google Chrome Browser on Linux

If you need a browser for accessing the Dashboard, here’s how to install **Google Chrome** on your Linux system.

### Step 1: Download Google Chrome
Run the following command to download the latest version of Google Chrome:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

If `wget` is not installed, you can install it using:

```bash
sudo apt install wget
```

### Step 2: Install Google Chrome
Install Chrome with the following command:

```bash
sudo apt install ./google-chrome-stable_current_amd64.deb
```

After installation, you may need to enter your user password.

---

## Conclusion
Congratulations! You've successfully set up the Kubernetes Dashboard on your existing Kubernetes cluster. With the Dashboard installed, you now have a powerful tool for monitoring, managing, and troubleshooting your Kubernetes workloads.

Explore the features of the Dashboard to streamline your Kubernetes operations and enhance your cluster management experience.

**Happy dashboarding!**
