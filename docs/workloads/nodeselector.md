# **Kubernetes Node Selectors: Practical Guide**

Node selectors are a straightforward way to schedule Kubernetes pods onto specific nodes based on labels. This guide provides a comprehensive background on node selectors, a practical example, and verification steps using the content provided.

---

## **1. Background on Node Selectors**

### **What Are Node Selectors?**
Node selectors are a Kubernetes feature that ensures pods are scheduled only on nodes that match specific labels. They provide a basic mechanism to assign pods to specific nodes without complex affinity rules.

### **Why Use Node Selectors?**
- **Workload Isolation**: Ensure specific workloads run only on designated nodes (e.g., staging vs. production environments).
- **Resource Optimization**: Schedule resource-intensive workloads on high-performance nodes.
- **Compliance**: Assign workloads to nodes with specific compliance or hardware requirements.

### **How Do They Work?**
1. Nodes in the cluster are labeled with `key=value` pairs.
2. In the pod spec, the `nodeSelector` field specifies the required labels for scheduling.
3. The Kubernetes scheduler places the pod only on nodes matching the specified labels.

---

## **2. Practical Example Using Node Selectors**

This example demonstrates node selectors with two worker nodes, `worker-node-1.example.com` and `worker-node-2.example.com`. The nodes will be labeled and pods will be scheduled accordingly.

---

### **Step 1: Label the Nodes**

Assign labels to the nodes to identify them for scheduling.

1. Label `worker-node-1.example.com` with `env=simplilearn`:
   ```bash
   kubectl label nodes worker-node-1.example.com env=simplilearn
   ```

2. Verify the labels:
   ```bash
   kubectl get nodes --show-labels
   ```

---

### **Step 2: Create a Pod Using Node Selectors**

1. Create a YAML file named `nodeselector.yaml`:
   ```bash
   nano nodeselector.yaml
   ```

2. Add the following content to the file:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx-labels
     labels:
       env: test
   spec:
     containers:
     - name: nginx
       image: nginx
       imagePullPolicy: IfNotPresent
     nodeSelector:
       env: simplilearn
   ```

3. Create the pod:
   ```bash
   kubectl create -f nodeselector.yaml
   ```

4. Verify the pod state:
   ```bash
   kubectl get pods -o wide
   ```

   - The pod should be scheduled on `worker-node-1.example.com`, as it matches the `env=simplilearn` label.

---

### **Step 3: Create a Pod Using Node Affinity (Advanced Use)**

Node selectors are basic, but you can use **node affinity** for more advanced scheduling policies. Here’s an example to avoid scheduling a pod on `worker-node-1` by using the `NotIn` operator.

1. Create a YAML file named `notin.yaml`:
   ```bash
   nano notin.yaml
   ```

2. Add the following content:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: with-node-affinity
   spec:
     affinity:
       nodeAffinity:
         preferredDuringSchedulingIgnoredDuringExecution:
         - weight: 1
           preference:
             matchExpressions:
             - key: env
               operator: NotIn
               values:
               - simplilearn
     containers:
     - name: httpd
       image: docker.io/httpd
   ```

3. Create the pod:
   ```bash
   kubectl create -f notin.yaml
   ```

4. Verify the pod state:
   ```bash
   kubectl get pods -o wide
   ```

   - The pod should be scheduled on `worker-node-2.example.com`, as the affinity rule excludes nodes labeled `env=simplilearn`.

---

### **Verification Steps**

1. **Check Node Labels**:
   ```bash
   kubectl get nodes --show-labels
   ```

   Ensure that the labels (`env=simplilearn`) are applied correctly to `worker-node-1`.

2. **Check Pod Scheduling**:
   ```bash
   kubectl get pods -o wide
   ```

   Verify that:
   - `nginx-labels` is running on `worker-node-1`.
   - `with-node-affinity` is running on `worker-node-2`.

3. **Debug Scheduling Issues (If Needed)**:
   ```bash
   kubectl describe pod <pod-name>
   ```

   Look for scheduling-related messages in the pod events section.

---

## **3. Summary**

| **Feature**          | **Description**                                                                 | **Example**                                                                                       |
|-----------------------|---------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| **Node Selectors**    | Schedule pods on nodes with specific labels.                                    | `nodeSelector: { env: simplilearn }` ensures the pod runs on nodes labeled with `env=simplilearn`.|
| **Node Affinity**     | Advanced scheduling to include or exclude nodes using logical operators.        | `NotIn` operator to exclude specific nodes.                                                     |
| **Pod Scheduling**    | Verify where pods are scheduled to confirm node selectors/affinity is working.  | `kubectl get pods -o wide` shows which node the pod is scheduled on.                             |

---

### **PDF with Example**

The provided example YAML files have been sourced from the attached PDF: *01_Configuring_Pods_with_Nodename_and_Nodeselector_Fields.pdf*. Follow the steps to ensure accurate implementation and validation of node selectors and affinity rules.

Let me know if you need further assistance!
