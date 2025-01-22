# **Kubernetes Scheduling Deep Dive**

This tutorial provides a practical guide to Kubernetes scheduling topics, covering performance tuning, policies, profiles, topology management, and constraints. Each section includes necessary background knowledge, practical examples, and use cases to help you apply these concepts.

---

## **1. Scheduler Performance Tuning**

### **Background**
The Kubernetes Scheduler handles assigning pods to nodes. As clusters grow, performance becomes critical. Fine-tuning the scheduler ensures efficient resource utilization and minimizes pod scheduling latency.

### **Key Tuning Parameters**
1. **Parallelism (`scheduler-threads`)**:
   - Controls how many scheduling operations can run concurrently.
   - Higher thread count increases throughput but uses more CPU.
   - Default: `16`.
   
2. **Cache Size**:
   - Configures the size of the scheduler cache, which stores nodes and pods information.
   - Affects how quickly the scheduler can respond to changes.

3. **Binding Timeout**:
   - Adjusts the time allowed for a pod to bind to a node.

### **Practical Example**
Tune the scheduler by passing arguments in the `kube-scheduler` deployment configuration.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-scheduler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: kube-scheduler
        image: k8s.gcr.io/kube-scheduler:v1.27.0
        command:
        - kube-scheduler
        args:
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --v=2
        - --scheduler-threads=32
```

### **Use Case**
High-throughput clusters (e.g., 1000+ nodes) where pod scheduling latency impacts performance.

---

## **2. Scheduling Policies**

### **Background**
Scheduling policies define rules and preferences for assigning pods to nodes. These ensure compliance with organizational requirements and optimize workloads.

### **Types of Policies**
1. **Node Affinity/Anti-Affinity**:
   - Example: Prefer nodes with specific labels like `ssd`.
2. **Taints and Tolerations**:
   - Prevent scheduling pods on certain nodes unless explicitly allowed.
3. **Custom Scheduling Plugins**:
   - Extend Kubernetes Scheduler with custom logic.

### **Example YAML**
#### Node Affinity Policy
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-demo
spec:
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
  - name: app
    image: nginx
```

### **Use Case**
- Workloads requiring specialized hardware like GPUs.
- Ensuring compliance by restricting workloads to specific zones or environments.

---

## **3. Scheduling Profiles**

### **Background**
Scheduling profiles allow you to define multiple scheduling configurations within the same cluster. Each profile can use different plugins for scheduling workflows.

### **Key Plugins in Profiles**
1. **DefaultBinder**: Binds pods to nodes.
2. **NodeResourcesFit**: Ensures sufficient resources are available.
3. **VolumeBinding**: Ensures volumes are bound before pod scheduling.

### **Practical Example**
Define two scheduling profiles in the `kube-scheduler` configuration.

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

### **Use Case**
- Separating scheduling workflows for regular and high-priority workloads.

---

## **4. Topology Management Policies**

### **Background**
Topology management policies ensure that pods are allocated resources in a way that optimizes hardware performance, particularly for NUMA (Non-Uniform Memory Access) architectures.

### **Available Policies**
1. **None**:
   - No resource alignment.
2. **BestEffort**:
   - Tries to align resources without strict guarantees.
3. **Restricted**:
   - Ensures aligned resources but may reduce scheduling flexibility.
4. **SingleNUMANode**:
   - Strictly aligns resources to a single NUMA node.

### **Practical Example**
Enable the `Restricted` policy in the kubelet configuration.

```yaml
apiVersion: kubelet.config.k8s.io/v1
kind: KubeletConfiguration
topologyManagerPolicy: "restricted"
```

### **Use Case**
- Workloads requiring low latency, such as AI/ML applications or real-time processing.

---

## **5. Pod Topology Spread Constraints**

### **Background**
Pod topology spread constraints distribute pods evenly across failure domains (e.g., zones, racks). This improves fault tolerance and resource utilization.

### **Key Parameters**
- **`topologyKey`**: The node label to spread across (e.g., `zone`).
- **`maxSkew`**: Maximum difference in pod count between failure domains.

### **Practical Example**
Spread pods evenly across zones.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spread-demo
spec:
  replicas: 6
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: demo
      containers:
      - name: app
        image: nginx
```

### **Use Case**
- Ensuring high availability by spreading pods across zones.
- Avoiding resource hotspots in specific racks or nodes.

---

## **Summary Table**

| **Topic**                  | **Description**                                                                                     | **Use Case**                                                                                      |
|----------------------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| **Scheduler Performance Tuning** | Optimize scheduler performance with parallelism and cache settings.                                 | Large-scale clusters requiring fast pod scheduling.                                             |
| **Scheduling Policies**    | Define rules for node affinity, tolerations, and custom logic.                                    | Specialized workloads like GPUs or compliance workloads.                                         |
| **Scheduling Profiles**    | Enable multiple scheduling workflows in a single cluster.                                         | Segregate scheduling for high-priority and regular workloads.                                   |
| **Topology Management Policies** | Align resources for hardware optimization (e.g., NUMA nodes).                                    | Low-latency applications like AI/ML or financial systems.                                       |
| **Pod Topology Spread Constraints** | Evenly distribute pods across failure domains.                                               | High availability and avoiding resource hotspots.                                               |

---

This comprehensive tutorial covers the core topics of Kubernetes scheduling with enough context to apply these concepts effectively in production environments.
