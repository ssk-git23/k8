# **Kubernetes Scheduler - Important Concepts in a Nutshell**

This tutorial provides a comprehensive guide to Kubernetes scheduling, including an overview of the Kubernetes Scheduler, its framework, node selection, pod allocation, and advanced concepts like scheduling profiles and topology management. It is designed to help you effectively manage workloads in a Kubernetes cluster.

---

## **1. Introduction to Kubernetes Scheduler**

The **Kubernetes Scheduler** is a core control-plane component responsible for assigning pods to appropriate nodes based on constraints and policies.

### **Key Responsibilities**
- Identify pods that lack assigned nodes.
- Evaluate available nodes against scheduling policies.
- Bind pods to the most suitable nodes.

### **Inputs to the Scheduler**
1. Pod specifications (resource requests, affinity rules, etc.).
2. Cluster state (nodes, resources, taints, and tolerations).

---

## **2. Scheduling Framework**

The Kubernetes Scheduler uses a **modular and extensible framework** to support custom scheduling needs.

### **Scheduling Workflow**
1. **PreFilter**: Ensures the pod meets initial conditions (e.g., resource requests, affinity/anti-affinity).
2. **Filter**: Excludes nodes that fail to meet the pod's requirements (e.g., taints, tolerations, resource availability).
3. **Score**: Assigns scores to remaining nodes based on suitability.
4. **Reserve**: Temporarily reserves resources on the selected node.
5. **Permit**: Optionally validates the node selection.
6. **Bind**: Finalizes the pod-to-node assignment.

### **Extensibility**
- Allows developers to build custom plugins for additional scheduling logic.

---

## **3. Node Selection**

Node selection is a critical process in scheduling, where nodes are evaluated for compatibility with the pod's requirements.

### **Key Factors**
1. **Node Conditions**: Nodes must be "Ready" and schedulable.
2. **Resource Availability**: Adequate CPU, memory, and other resources.
3. **Node Affinity/Anti-Affinity**: Prefer or avoid specific nodes.
4. **Taints and Tolerations**: Control pod placement on tainted nodes.
5. **Topology Spread Constraints**: Ensure even distribution of pods across failure domains.

---

## **4. Pod Allocation**

Pod allocation finalizes scheduling by binding the selected node to the pod.

### **Steps in Pod Allocation**
1. **Admission Control**: Validates the pod against cluster policies.
2. **Scoring**: Ranks nodes using scoring plugins.
3. **Binding**: Assigns the pod to the top-scoring node.

### **Constraints**
- **Resource Requests and Limits**: Ensure the node can meet pod resource demands.
- **Pod Affinity/Anti-Affinity**: Influence placement based on existing pods.

---

## **5. Scheduling Policies**

Scheduling policies define rules and preferences for workload placement.

### **Common Policies**
1. **Node Affinity**: Place pods on nodes with specific labels.
2. **Taints and Tolerations**: Restrict pods to specific nodes.
3. **Pod Affinity/Anti-Affinity**: Place pods near or away from other pods.

---

## **6. Advanced Scheduling Features**

### **Scheduling Profiles**
- Allow multiple scheduling configurations within the same cluster.
- Useful for separating workflows for high-priority and regular workloads.

Example:
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: "default-scheduler"
  plugins:
    score:
      enabled:
      - name: NodeResourcesBalancedAllocation
- schedulerName: "high-priority-scheduler"
  plugins:
    score:
      enabled:
      - name: InterPodAffinity
```

### **Topology Management**
- Ensures resource alignment for performance-sensitive workloads (e.g., AI/ML).
- Policies:
  - **None**: No alignment.
  - **Restricted**: Ensures resource alignment but may reduce flexibility.
  - **SingleNUMANode**: Strict alignment for NUMA nodes.

---

## **7. Practical Example: Pod Scheduling**

### **Node Preparation**
1. Label a node:
   ```bash
   kubectl label nodes node1 disktype=ssd
   ```
2. Taint a node:
   ```bash
   kubectl taint nodes node1 key=value:NoSchedule
   ```

### **Pod YAML Example**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: scheduling-demo
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "disktype"
            operator: In
            values:
            - "ssd"
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
```

### **Commands**
1. Apply the manifest:
   ```bash
   kubectl apply -f pod.yaml
   ```
2. Verify the pod’s node:
   ```bash
   kubectl get pods -o wide
   ```

---

## **8. Summary**

| **Component**           | **Description**                                                   |
|-------------------------|-------------------------------------------------------------------|
| **Scheduler**           | Assigns pods to nodes based on constraints and policies.         |
| **Scheduling Framework**| Extensible phases for custom scheduling logic.                   |
| **Node Selection**       | Evaluates node conditions, resources, and topology constraints. |
| **Pod Allocation**       | Finalizes pod-to-node binding.                                  |
| **Scheduling Policies**  | Defines rules for affinity, taints, and custom logic.           |

---

## **Use Cases**
1. **High Availability**: Spread workloads across failure domains.
2. **Specialized Workloads**: Allocate resources for GPUs, NUMA nodes, etc.
3. **Workload Isolation**: Use taints and tolerations for isolating workloads.

This guide equips you with a solid understanding of Kubernetes scheduling, enabling you to optimize cluster performance and ensure efficient resource utilization.
