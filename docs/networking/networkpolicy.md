### **Lab Tutorial: NetworkPolicy with Allow and Deny-All Ingress**

This tutorial provides an introduction to **Kubernetes NetworkPolicy**, discusses its use cases and dependencies on container networking interfaces (CNIs) like Calico and Flannel, and demonstrates how to configure "allow" and "deny-all" ingress rules.

---

### **What is a NetworkPolicy?**

A **NetworkPolicy** in Kubernetes is a resource used to define rules for network traffic to and from Pods. By default, Kubernetes allows all traffic to flow between Pods, Nodes, and external services. NetworkPolicies enable you to:
- **Restrict incoming (ingress) or outgoing (egress) traffic** to specific Pods.
- Define rules based on Pod labels, namespaces, and IP blocks.
- Enhance security by isolating workloads in a multi-tenant or microservices architecture.

---

### **Use Cases for NetworkPolicy**
1. **Application Isolation**: Ensure one application cannot unintentionally communicate with another.
2. **Compliance**: Enforce strict network segmentation to meet regulatory requirements.
3. **Traffic Restriction**: Control access to applications from specific sources, like trusted Pods or IP addresses.
4. **Improved Security**: Prevent unauthorized traffic to sensitive components, such as APIs or databases.

---

### **Dependencies: Container Networking Interface (CNI)**
NetworkPolicies depend on the underlying **CNI plugin** to enforce traffic rules. Kubernetes itself does not enforce NetworkPolicies. Supported CNIs include:

1. **Calico**:
   - Full support for NetworkPolicies.
   - Advanced features like global policies, service-based policies, and BGP routing.
   - Best suited for production-grade deployments.

2. **Flannel**:
   - Does not support NetworkPolicies out-of-the-box.
   - Requires **Flannel with Calico** to enable NetworkPolicy enforcement.

3. **Cilium**:
   - Focuses on security and observability.
   - Offers fine-grained control and deep integration with Linux BPF.

4. **Weave Net**:
   - Supports basic NetworkPolicies.
   - Simplified setup for smaller-scale clusters.

---

### **Prerequisites**

1. Kubernetes cluster is up and running.
2. A CNI plugin that supports NetworkPolicies (e.g., Calico).
   - To install Calico:
     ```bash
     kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
     ```
3. `kubectl` command-line tool.

---

### **Section 1: Allow Ingress Traffic**

#### **1. Create the Main Application Pod**
Launch the main application pod with the `app=simplilearn` label:
```bash
kubectl run simplilearn --image=nginx --labels="app=simplilearn" --expose --port=80
```
Verify the pod and service:
```bash
kubectl get pods
kubectl get svc simplilearn
```

#### **2. Create an Allow-Ingress NetworkPolicy**
Create a YAML file named `allow-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: simplilearn-allow-ingress
spec:
  podSelector:
    matchLabels:
      app: simplilearn
  ingress:
  - from:
      - podSelector:
          matchLabels:
            app: simplilearn
```

Apply the NetworkPolicy:
```bash
kubectl apply -f allow-ingress.yaml
```

Verify the policy:
```bash
kubectl get networkpolicy
```

#### **3. Test Allow Ingress**
Run a testing pod **with matching labels**:
```bash
kubectl run --image=nginx test-$RANDOM --labels="app=simplilearn"
kubectl exec -it test-<podname> -- wget -qO- http://simplilearn
```

Expected Result:
- Traffic is **allowed**, and the NGINX default page should load.

---

### **Section 2: Deny-All Ingress Traffic**

#### **1. Create a Deny-All NetworkPolicy**
Create a YAML file named `deny-all-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: simplilearn-deny-all
spec:
  podSelector:
    matchLabels:
      app: simplilearn
  ingress: []
```

Apply the policy:
```bash
kubectl apply -f deny-all-ingress.yaml
```

Verify the policy:
```bash
kubectl get networkpolicy
```

#### **2. Test Deny-All Ingress**
Run a testing pod to verify blocked traffic:
```bash
kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
wget -qO- http://simplilearn
```

Expected Result:
- Traffic is **blocked**, and the connection times out.

---

### **Illustration: Comparing Allow and Deny Policies**

1. **Allow Policy**:
   - Test with a matching label:
     ```bash
     wget -qO- http://simplilearn
     ```
   - Result: NGINX default page loads, indicating **allowed ingress**.

2. **Deny-All Policy**:
   - Test with any Pod, regardless of labels:
     ```bash
     wget -qO- http://simplilearn
     ```
   - Result: Traffic is **blocked**, and the connection times out.

---

### **Cleanup**

1. Delete the application pod:
   ```bash
   kubectl delete pod simplilearn
   ```

2. Delete the service:
   ```bash
   kubectl delete svc simplilearn
   ```

3. Delete the NetworkPolicies:
   ```bash
   kubectl delete networkpolicy simplilearn-allow-ingress
   kubectl delete networkpolicy simplilearn-deny-all
   ```

---

### **Summary**

- **NetworkPolicies** allow you to control ingress and egress traffic to Pods, enhancing security and traffic management.
- Dependencies like Calico or Cilium are necessary to enforce policies.
- This tutorial demonstrated the configuration and testing of both "allow" and "deny-all" ingress rules, providing a clear understanding of their impact. 

Let me know if you need further assistance!
