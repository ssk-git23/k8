# Lab Tutorial: Taints and Tolerations in Kubernetes

## Table of Contents
1. [**Overview of Taints and Tolerations**](#overview-of-taints-and-tolerations)
2. [**Setting Up the Lab Environment**](#setting-up-the-lab-environment)
3. [**Applying Taints to All Worker Nodes**](#applying-taints-to-all-worker-nodes)
4. [**Deploying a Pod Without Tolerations**](#deploying-a-pod-without-tolerations)
5. [**Adding Tolerations to the Pod**](#adding-tolerations-to-the-pod)
6. [**Validating Pod Scheduling**](#validating-pod-scheduling)
7. [**Use Cases/Benefits, Summary and Key Takeaways**](#summary-and-key-takeaways)

---

## 1. Overview of Taints and Tolerations

Taints and tolerations allow control over pod scheduling:
- **Taints** repel pods from nodes.
- **Tolerations** allow pods to override taints and schedule on tainted nodes.

Use Cases:
- Dedicated nodes for specific workloads.
- Prevent pods from being scheduled during maintenance or upgrades.
- Enforce node isolation for critical applications.

---

## 2. Setting Up the Lab Environment

Confirm your Kubernetes cluster setup with two worker nodes (`worker-node-1`, `worker-node-2`) and one master node (`master`). Verify the node names:

```bash
kubectl get nodes
```

Output:
```plaintext
NAME              STATUS   ROLES           AGE   VERSION
master            Ready    control-plane   10d   v1.26
worker-node-1     Ready    <none>          10d   v1.26
worker-node-2     Ready    <none>          10d   v1.26
```

---

## 3. Applying Taints to All Worker Nodes

Taint both `worker-node-1` and `worker-node-2` to ensure pods without tolerations cannot be scheduled on either node.

### **Commands to Apply Taints**
For `worker-node-1`:
```bash
kubectl taint nodes worker-node-1 key=example-taint:NoSchedule
```

For `worker-node-2`:
```bash
kubectl taint nodes worker-node-2 key=example-taint:NoSchedule
```

### **Verify Taints**
Check that taints are applied to both nodes:

```bash
kubectl describe node worker-node-1 | grep -i taints
kubectl describe node worker-node-2 | grep -i taints
```

Expected output:
```plaintext
Taints: key=example-taint:NoSchedule
```

---

## 4. Deploying a Pod Without Tolerations

### **Pod YAML Without Tolerations**

Create the following YAML file (`no-tolerations-pod.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-tolerations-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Apply the configuration:

```bash
kubectl apply -f no-tolerations-pod.yaml
```

### **Expected Behavior**

Check the pod's status:

```bash
kubectl get pods -o wide
```

The pod will remain in a `Pending` state because both worker nodes are tainted, and the pod cannot tolerate the taints:

```plaintext
NAME                  READY   STATUS    NODE
no-tolerations-pod    0/1     Pending   <none>
```

### **Describe the Pod**

To confirm the scheduling issue, describe the pod:

```bash
kubectl describe pod no-tolerations-pod
```

Look for the `Events` section, which will show a message like:
```plaintext
0/2 nodes are available: 2 node(s) had taint {key: example-taint}, that the pod didn't tolerate.
```

---

## 5. Adding Tolerations to the Pod

### **Pod YAML With Tolerations**

Create a new YAML file (`tolerations-pod.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerations-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "example-taint"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Explanation:
- **`key`, `value`**: Match the taint key-value pair on the nodes.
- **`effect`**: `NoSchedule` ensures the toleration aligns with the node taint.

Apply the updated pod configuration:

```bash
kubectl apply -f tolerations-pod.yaml
```

---

## 6. Validating Pod Scheduling

### **Check Pod Status**

Verify that the pod is now running:

```bash
kubectl get pods -o wide
```

Expected output:
```plaintext
NAME               READY   STATUS    NODE
tolerations-pod    1/1     Running   worker-node-1
```

If `worker-node-1` is busy, the pod will be scheduled on `worker-node-2`.

### **Describe the Pod**

To ensure the tolerations are applied correctly, describe the pod:

```bash
kubectl describe pod tolerations-pod
```

Check the `Tolerations` section in the output to confirm the pod tolerates the taint.

---

## 7. Use Cases/Benefits, Summary and Key Takeaways

### **Use Cases of Taints and Tolerations**

1. **Node Isolation for Specific Workloads**  
   - Taints allow nodes to be reserved for specific workloads, such as GPU-heavy tasks or sensitive applications.
   - **Example**: A node with specialized hardware (e.g., GPU or FPGA) is tainted so only machine-learning or high-performance workloads with matching tolerations can run.

2. **Maintenance or Upgrades**  
   - During node maintenance or upgrades, taints can be applied to prevent new pods from being scheduled while ensuring existing pods are not evicted unless necessary.  
   - **Example**: Temporarily taint a node (`key=maintenance:NoSchedule`) so no new pods are placed there, but existing critical services remain unaffected if tolerations are pre-configured.

3. **Critical Applications Placement**  
   - Taints ensure that nodes dedicated to critical services only allow pods with specific tolerations.
   - **Example**: Database servers or monitoring tools are deployed only on specific, tainted nodes.

4. **Cluster Workload Segmentation**  
   - Divide cluster workloads by applying taints to certain node pools based on workload type or priority.  
   - **Example**: Development and production workloads are isolated by tainting the production node pool (`key=production:NoSchedule`) and allowing only toleration-configured production pods to schedule there.

5. **High Availability and Failover**  
   - In a multi-cluster setup, taints can prevent pods from failing over to unsuitable nodes or clusters during outages.  
   - **Example**: In disaster recovery, taints prevent failover to nodes that are not pre-configured for certain workloads.

---

### **Benefits of Taints and Tolerations**

1. **Enhanced Control Over Pod Scheduling**  
   - Precisely define where specific workloads should or should not run.
   - Prevent accidental scheduling of unwanted workloads on specialized or maintenance nodes.

2. **Node Resource Optimization**  
   - Reserve resources on nodes for high-priority or critical applications, avoiding competition from general-purpose workloads.

3. **Cluster Policy Enforcement**  
   - Enforce strict workload placement policies to meet organizational or compliance requirements.

4. **Improved Fault Isolation**  
   - Protect critical workloads by isolating them on specific nodes with appropriate tolerations.

5. **Flexibility and Scalability**  
   - Enable dynamic scheduling rules for workloads by adjusting taints and tolerations as workloads evolve or requirements change.

---

### **Real-Time Examples**

1. **Isolating GPU Nodes for AI Workloads**  
   - A GPU node is tainted with `key=gpu:NoSchedule`. Only AI workloads with tolerations can run on this node, preventing resource wastage by general-purpose workloads.

2. **Node Upgrades Without Downtime**  
   - Apply a taint `key=upgrade:NoSchedule` to a node undergoing OS or Kubernetes version updates. Non-tolerating pods are not scheduled, but tolerating critical workloads remain operational.

3. **Production Workload Isolation**  
   - Nodes hosting production workloads are tainted with `key=env:production:NoSchedule`. Only production pods with matching tolerations can be deployed, ensuring environment segmentation.

4. **Regional Node Failover Prevention**  
   - Nodes in a specific region are tainted with `key=region:us-west:NoSchedule`. Workloads from other regions cannot failover to this region unless tolerations are defined.

5. **Testing and Staging Environments**  
   - Nodes used for testing are tainted with `key=testing:NoSchedule`. Only pods related to testing (with tolerations) can run, preventing interference from production workloads.

---

These examples, benefits, and use cases highlight how taints and tolerations can be used to manage workloads effectively in real-world Kubernetes environments.

### **What We Learned**
1. Taints repel pods from nodes unless explicitly tolerated.
2. Tolerations allow pods to override taints and schedule on tainted nodes.

### **Use Cases**
- Reserve nodes for critical workloads or GPU-based tasks.
- Prevent general workloads during node maintenance or upgrade operations.

### **Verification Steps**
1. Apply taints to all worker nodes to repel pods without tolerations.
2. Confirm that pods without tolerations remain in the `Pending` state.
3. Add tolerations to the pod and validate that it gets scheduled on a tainted node.

This tutorial ensures a clear understanding of how taints and tolerations work in real-world scenarios, with verification steps to validate pod scheduling behavior.
