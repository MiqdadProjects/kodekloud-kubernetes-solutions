**🌟 Task 3 - Create Namespace and Nginx Pod**

**📌 Task Description**

The Nautilus DevOps team is setting up Kubernetes namespaces and deployments for application migration. A team member has been tasked with creating a namespace and deploying a pod within it.

**Requirements:**
- Create a namespace named `dev`.
- Create a pod named `dev-nginx-pod` in the `dev` namespace using the `nginx:latest` image (specify the tag).
- Label the pod with `app: dev-nginx-pod`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like namespaces or pod names may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Set up a namespace and deploy an nginx pod within it.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Namespace and Pod Manifest

```bash
vi dev-nginx-pod.yaml
```

**Purpose**: Create a YAML file defining the namespace and pod.

**YAML Content**:
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx-pod
  namespace: dev
  labels:
    app: dev-nginx-pod
spec:
  containers:
    - name: dev-nginx-pod
      image: nginx:latest
      imagePullPolicy: IfNotPresent
  restartPolicy: Always
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Namespace`: Creates the `dev` namespace.
- `metadata.name`: Sets namespace name to `dev`.
- `kind: Pod`: Defines a pod in the `dev` namespace.
- `metadata.namespace`: Places the pod in the `dev` namespace.
- `metadata.labels`: Adds `app: dev-nginx-pod` label.
- `spec.containers`: Uses `nginx:latest` image.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.
- `restartPolicy: Always`: Ensures pod restarts.

---

## 🔹 Step 3: Apply the Manifest

```bash
kubectl apply -f dev-nginx-pod.yaml
```

**Purpose**: Create the namespace and pod.

**Expected Output**:
```
namespace/dev created
pod/dev-nginx-pod created
```

---

## 🔹 Step 4: Verify Namespace

```bash
kubectl get namespaces
```

**Purpose**: Confirm the `dev` namespace exists.

**Expected Output**:
```
NAME              STATUS   AGE
dev               Active   10s
...
```

---

## 🔹 Step 5: Verify Pod

```bash
kubectl get pods -n dev
```

**Purpose**: Confirm the pod is running in the `dev` namespace.

**Expected Output**:
```
NAME             READY   STATUS    RESTARTS   AGE
dev-nginx-pod   1/1     Running   0          10s
```

**Verify Labels**:
```bash
kubectl get pods -n dev -l app=dev-nginx-pod
```

**Expected Output**:
```
NAME             READY   STATUS    RESTARTS   AGE
dev-nginx-pod   1/1     Running   0          10s
```

---

## 🔹 Step 6: Verify Pod Details

```bash
kubectl describe pod dev-nginx-pod -n dev
```

**Purpose**: Check pod details, including namespace and labels.

**Expected Output (partial)**:
```
Name:         dev-nginx-pod
Namespace:    dev
Labels:       app=dev-nginx-pod
Containers:
  dev-nginx-pod:
    Image:          nginx:latest
    State:          Running
```

---

## 📋 Quick Command Reference

```bash
# Create and apply manifest
vi dev-nginx-pod.yaml
# (Paste YAML content)
kubectl apply -f dev-nginx-pod.yaml

# Verify namespace and pod
kubectl get namespaces
kubectl get pods -n dev
kubectl get pods -n dev -l app=dev-nginx-pod
kubectl describe pod dev-nginx-pod -n dev
```

---

## 💡 Additional Tips

- **Task Variability**: Namespace or pod names may differ; update the YAML as needed.
- **Namespace Isolation**: Use namespaces to segregate resources in multi-tenant clusters.
- **Label Consistency**: Ensure labels are unique within the namespace.
- **Dry Run**: Test with `kubectl apply -f dev-nginx-pod.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Namespace Not Found**
**Symptoms**: `Error: namespace "dev" not found`.
**Solution**: Create the namespace manually if missing.
```bash
kubectl create namespace dev
```

### **Issue 2: Pod Not Running**
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events.
```bash
kubectl describe pod dev-nginx-pod -n dev
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod dev-nginx-pod -n dev
# Ensure image: nginx:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **creating a pod within a specific namespace** with correct labeling and image configuration.

**💡 Solution Approach:**
1. **Namespace Creation**: Defined the `dev` namespace in the YAML.
2. **Pod Configuration**: Specified `namespace: dev` and `nginx:latest` image.
3. **Labeling**: Added `app: dev-nginx-pod` for identification.
4. **Verification**: Used namespace-scoped `kubectl` commands to confirm.

**🎯 Key Success Factors:**
- **Namespace isolation**: Ensured pod is created in `dev` namespace.
- **Correct labeling**: Applied `app: dev-nginx-pod`.
- **Image tag**: Used `nginx:latest` explicitly.
- **Verification**: Confirmed pod status and namespace.

**⚠️ Critical Configuration Details:**
- **Namespace**: Must be `dev`.
- **Pod Name**: Must be `dev-nginx-pod`.
- **Image**: Must be `nginx:latest`.
- **Label**: Must include `app: dev-nginx-pod`.

**🔒 Kubernetes Benefits:**
- **Isolation**: Namespaces provide resource segregation.
- **Reliability**: `restartPolicy: Always` ensures pod recovery.
- **Organization**: Labels enable resource filtering.

---

## ⚠️ Important Production Notes

🔧 **Namespace Usage**: Organizes resources for multi-tenant environments.

🔐 **Security**: Consider RBAC for namespace access control in production.

📊 **Monitoring**: Monitor namespace resource usage with tools like Prometheus.

🛡️ **Scalability**: Use Deployments for scaling instead of standalone pods.

---