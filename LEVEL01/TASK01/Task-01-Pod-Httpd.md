**🌟 Task 1 - Create HTTPD Pod**

**📌 Task Description**

The Nautilus DevOps team is initiating Kubernetes adoption for application migration. A team member has been tasked with creating a pod to test basic deployment capabilities.

**Requirements:**
- Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag (specify `httpd:latest`).
- Set the label `app: httpd_app`.
- Name the container `httpd-container`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names or namespaces may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy a simple HTTPD pod with specified labels and container name.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host where `kubectl` is configured to interact with the Kubernetes cluster.

---

## 🔹 Step 2: Create Pod Manifest

```bash
vi pod-httpd.yaml
```

**Purpose**: Create a YAML file to define the pod configuration.

**YAML Content**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      imagePullPolicy: IfNotPresent
  restartPolicy: Always
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `apiVersion: v1`: Specifies the Kubernetes API version for pods.
- `kind: Pod`: Defines the resource type as a pod.
- `metadata.name`: Sets the pod name to `pod-httpd`.
- `metadata.labels`: Adds the `app: httpd_app` label.
- `spec.containers`: Defines the container with name `httpd-container` and image `httpd:latest`.
- `imagePullPolicy: IfNotPresent`: Pulls the image only if not already present locally.
- `restartPolicy: Always`: Ensures the pod restarts on failure.

---

## 🔹 Step 3: Apply the Pod Manifest

```bash
kubectl apply -f pod-httpd.yaml
```

**Purpose**: Deploy the pod to the Kubernetes cluster.

**Expected Output**:
```
pod/pod-httpd created
```

---

## 🔹 Step 4: Verify Pod Creation

```bash
kubectl get pods
```

**Purpose**: Confirm the pod is running.

**Expected Output**:
```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          10s
```

**Additional Verification**:
```bash
kubectl describe pod pod-httpd
```

**Purpose**: Check pod details, including labels and container status.

**Expected Output (partial)**:
```
Name:         pod-httpd
Labels:       app=httpd_app
Containers:
  httpd-container:
    Image:          httpd:latest
    State:          Running
```

---

## 🔹 Step 5: Verify Label

```bash
kubectl get pods -l app=httpd_app
```

**Purpose**: Ensure the pod has the correct label.

**Expected Output**:
```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          1m
```

---

## 📋 Quick Command Reference

```bash
# Create and apply pod manifest
vi pod-httpd.yaml
# (Paste YAML content)
kubectl apply -f pod-httpd.yaml

# Verify pod and labels
kubectl get pods
kubectl get pods -l app=httpd_app
kubectl describe pod pod-httpd
```

---

## 💡 Additional Tips

- **Task Variability**: Pod names or namespaces may differ in your environment; adapt the YAML accordingly.
- **Image Tag**: Always specify `latest` explicitly to avoid ambiguity.
- **Labels**: Use consistent labeling for easy resource filtering.
- **kubectl Dry Run**: Test YAML syntax with `kubectl apply -f pod-httpd.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod in Pending State**
**Symptoms**: `kubectl get pods` shows `Pending` status.
**Solution**: Check cluster resources and events.
```bash
kubectl describe pod pod-httpd
# Look for events indicating resource issues
```

### **Issue 2: Image Pull Failure**
**Symptoms**: `ErrImagePull` or `ImagePullBackOff` status.
**Solution**: Verify image name and connectivity.
```bash
kubectl describe pod pod-httpd
# Ensure image: httpd:latest is correct
```

### **Issue 3: Incorrect Labels**
**Symptoms**: `kubectl get pods -l app=httpd_app` returns no pods.
**Solution**: Verify pod labels.
```bash
kubectl get pods pod-httpd --show-labels
kubectl label pod pod-httpd app=httpd_app --overwrite
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **creating a pod with precise naming and labeling** to meet the requirements while ensuring the pod runs successfully in the Kubernetes cluster.

**💡 Solution Approach:**
1. **Precise YAML Configuration**: Defined the pod with exact names (`pod-httpd`, `httpd-container`) and label (`app: httpd_app`).
2. **Image Specification**: Used `httpd:latest` with explicit tag to avoid ambiguity.
3. **Verification Steps**: Used `kubectl get` and `describe` to confirm pod status and labels.
4. **Best Practices**: Set `imagePullPolicy: IfNotPresent` to optimize image pulling and `restartPolicy: Always` for reliability.

**🎯 Key Success Factors:**
- **Accurate naming**: Ensured pod and container names matched requirements.
- **Label consistency**: Applied the required `app: httpd_app` label.
- **Status verification**: Confirmed pod is in `Running` state.
- **Efficient YAML**: Kept the manifest minimal yet complete.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `pod-httpd`.
- **Container Name**: Must be `httpd-container`.
- **Image**: Must be `httpd:latest`.
- **Label**: Must include `app: httpd_app`.
- **Cluster Access**: Assumes `kubectl` is configured on the jump host.

**🔒 Kubernetes Benefits:**
- **Simplicity**: Single-container pod for straightforward deployment.
- **Reliability**: `restartPolicy: Always` ensures pod recovery.
- **Scalability**: Labels enable future integration with services or controllers.

---

## ⚠️ Important Production Notes

🔧 **Pod Deployment**: Deploys a single-container pod suitable for basic web serving.

🔐 **Security Considerations**: Use `imagePullPolicy: IfNotPresent` to avoid unnecessary pulls; consider private registries for production.

📊 **Resource Management**: Monitor pod resource usage to prevent cluster overload.

🛡️ **Scalability**: Use ReplicaSets or Deployments for scaling in production.

---
