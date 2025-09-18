**🌟 Task 4 - Create Pod with Resource Limits**

**📌 Task Description**

The Nautilus DevOps team is addressing performance issues by setting resource limits for applications. A team member has been tasked with creating a pod with specific CPU and memory constraints.

**Requirements:**
- Create a pod named `httpd-pod` with container `httpd-container` using `httpd:latest` image.
- Set resource requests: Memory: 15Mi, CPU: 100m.
- Set resource limits: Memory: 20Mi, CPU: 100m.
- Label the pod with `app: httpd-pod`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names or resource values may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy an httpd pod with defined resource constraints.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Pod Manifest

```bash
vi httpd-pod.yaml
```

**Purpose**: Define the pod with resource limits and requests.

**YAML Content**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          cpu: "100m"
          memory: "20Mi"
        requests:
          cpu: "100m"
          memory: "15Mi"
  restartPolicy: Always
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `resources.limits`: Caps CPU at 100 milliCPU and memory at 20MiB.
- `resources.requests`: Reserves 100 milliCPU and 15MiB memory.
- `cpu: 100m`: 100 milliCPU (0.1 CPU core).
- `memory: 20Mi/15Mi`: Memory in mebibytes.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.
- `restartPolicy: Always`: Ensures pod restarts.

---

## 🔹 Step 3: Apply the Pod Manifest

```bash
kubectl apply -f httpd-pod.yaml
```

**Purpose**: Deploy the pod with resource constraints.

**Expected Output**:
```
pod/httpd-pod created
```

---

## 🔹 Step 4: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm the pod is running.

**Expected Output**:
```
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          10s
```

---

## 🔹 Step 5: Verify Resource Limits

```bash
kubectl describe pod httpd-pod
```

**Purpose**: Check resource limits and requests.

**Expected Output (partial)**:
```
Name:         httpd-pod
Labels:       app=httpd-pod
Containers:
  httpd-container:
    Image:          httpd:latest
    Limits:
      cpu:          100m
      memory:       20Mi
    Requests:
      cpu:          100m
      memory:       15Mi
    State:          Running
```

---

## 🔹 Step 6: Verify Labels

```bash
kubectl get pods -l app=httpd-pod
```

**Purpose**: Ensure the pod has the correct label.

**Expected Output**:
```
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          1m
```

---

## 📋 Quick Command Reference

```bash
# Create and apply pod manifest
vi httpd-pod.yaml
# (Paste YAML content)
kubectl apply -f httpd-pod.yaml

# Verify pod and resources
kubectl get pods
kubectl describe pod httpd-pod
kubectl get pods -l app=httpd-pod
```

---

## 💡 Additional Tips

- **Task Variability**: Resource values or pod names may differ; adjust YAML as needed.
- **Resource Limits**: Set limits slightly higher than requests to allow flexibility.
- **Monitoring**: Use tools like Prometheus to track resource usage.
- **Dry Run**: Test with `kubectl apply -f httpd-pod.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Evicted**
**Symptoms**: Pod status shows `Evicted` due to resource constraints.
**Solution**: Check cluster resource availability.
```bash
kubectl describe pod httpd-pod
# Increase cluster resources or adjust limits
```

### **Issue 2: Invalid Resource Syntax**
**Symptoms**: `kubectl apply` fails with syntax error.
**Solution**: Validate YAML syntax.
```bash
kubectl apply -f httpd-pod.yaml --dry-run=client -o yaml
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod httpd-pod
# Ensure image: httpd:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **configuring precise resource limits and requests** to ensure the pod runs within specified CPU and memory constraints.

**💡 Solution Approach:**
1. **Resource Configuration**: Defined `requests` and `limits` for CPU and memory.
2. **Accurate YAML**: Used `httpd:latest` and correct pod/container names.
3. **Verification**: Confirmed resource settings with `kubectl describe`.
4. **Best Practices**: Set `imagePullPolicy: IfNotPresent` for efficiency.

**🎯 Key Success Factors:**
- **Resource accuracy**: Set CPU to 100m and memory to 15Mi/20Mi.
- **Labeling**: Applied `app: httpd-pod` for identification.
- **Pod status**: Ensured `Running` state.
- **Minimal YAML**: Kept configuration concise.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `httpd-pod`.
- **Container Name**: Must be `httpd-container`.
- **Image**: Must be `httpd:latest`.
- **Resources**: Must match specified requests and limits.

**🔒 Kubernetes Benefits:**
- **Resource Control**: Prevents resource overuse.
- **Predictability**: Ensures consistent performance.
- **Scalability**: Prepares for cluster resource management.

---

## ⚠️ Important Production Notes

🔧 **Resource Management**: Limits ensure fair resource allocation.

🔐 **Security**: Use resource quotas in production namespaces.

📊 **Monitoring**: Track CPU/memory usage with monitoring tools.

🛡️ **Optimization**: Adjust limits based on application needs.

---