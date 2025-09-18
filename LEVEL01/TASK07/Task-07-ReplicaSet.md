**🌟 Task 7 - Create HTTPD ReplicaSet**

**📌 Task Description**

The Nautilus DevOps team is deploying applications requiring high availability. A team member has been tasked with creating a ReplicaSet to ensure multiple pod instances.

**Requirements:**
- Create a ReplicaSet named `httpd-replicaset` using `httpd:latest` image.
- Set labels: `app: httpd_app`, `type: front-end`.
- Name the container `httpd-container`.
- Ensure 4 replicas.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like ReplicaSet names or labels may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy a ReplicaSet with 4 httpd pods.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create ReplicaSet Manifest

```bash
vi httpd-replicaset.yaml
```

**Purpose**: Define the ReplicaSet with 4 replicas.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset
  labels:
    app: httpd_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: httpd-container
          image: httpd:latest
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: ReplicaSet`: Defines a ReplicaSet resource.
- `metadata.labels`: Sets `app: httpd_app`, `type: front-end`.
- `spec.replicas`: Ensures 4 pod instances.
- `spec.selector`: Matches pods with `tier: frontend` label.
- `spec.template`: Defines pod template with `httpd:latest` image.
- `imagePullPolicy`: Defaults to `IfNotPresent`.

---

## 🔹 Step 3: Apply the ReplicaSet Manifest

```bash
kubectl apply -f httpd-replicaset.yaml
```

**Purpose**: Deploy the ReplicaSet.

**Expected Output**:
```
replicaset.apps/httpd-replicaset created
```

---

## 🔹 Step 4: Verify ReplicaSet

```bash
kubectl get replicaset
```

**Purpose**: Confirm the ReplicaSet is created with 4 replicas.

**Expected Output**:
```
NAME               DESIRED   CURRENT   READY   AGE
httpd-replicaset   4         4         4       10s
```

---

## 🔹 Step 5: Verify Pods

```bash
kubectl get pods -l tier=frontend
```

**Purpose**: Ensure 4 pods are running.

**Expected Output**:
```
NAME                     READY   STATUS    RESTARTS   AGE
httpd-replicaset-xxx   1/1     Running   0          10s
httpd-replicaset-xxx   1/1     Running   0          10s
httpd-replicaset-xxx   1/1     Running   0          10s
httpd-replicaset-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 6: Verify ReplicaSet Details

```bash
kubectl describe replicaset httpd-replicaset
```

**Purpose**: Check ReplicaSet configuration and pod status.

**Expected Output (partial)**:
```
Name:         httpd-replicaset
Selector:     tier=frontend
Labels:       app=httpd_app
              type=front-end
Replicas:     4 current / 4 desired
Containers:
  httpd-container:
    Image:        httpd:latest
```

---

## 📋 Quick Command Reference

```bash
# Create and apply ReplicaSet manifest
vi httpd-replicaset.yaml
# (Paste YAML content)
kubectl apply -f httpd-replicaset.yaml

# Verify ReplicaSet and pods
kubectl get replicaset
kubectl get pods -l tier=frontend
kubectl describe replicaset httpd-replicaset
```

---

## 💡 Additional Tips

- **Task Variability**: ReplicaSet names or labels may differ; adjust YAML as needed.
- **Replicas**: Ensure cluster has enough resources for 4 pods.
- **Labels**: Match `selector` and `template` labels (`tier: frontend`).
- **Dry Run**: Test with `kubectl apply -f httpd-replicaset.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Fewer Pods Than Desired**
**Symptoms**: `kubectl get replicaset` shows fewer than 4 pods.
**Solution**: Check cluster resources.
```bash
kubectl describe replicaset httpd-replicaset
# Look for resource shortage events
```

### **Issue 2: Label Mismatch**
**Symptoms**: Pods not managed by ReplicaSet.
**Solution**: Verify `selector` and `template` labels.
```bash
kubectl get pods -l tier=frontend
# Fix labels if mismatched
```

### **Issue 3: Image Pull Failure**
**Symptoms**: Pods in `ErrImagePull` state.
**Solution**: Verify image.
```bash
kubectl describe pod httpd-replicaset-xxx
# Ensure image: httpd:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **ensuring 4 replicas** with correct labeling and container configuration.

**💡 Solution Approach:**
1. **ReplicaSet Configuration**: Defined 4 replicas with `httpd:latest`.
2. **Label Consistency**: Used `tier: frontend` for selector and pod template.
3. **Verification**: Confirmed 4 pods running with correct labels.
4. **Best Practices**: Kept YAML minimal and used standard labels.

**🎯 Key Success Factors:**
- **Replica count**: Set to 4 as required.
- **Label accuracy**: `app: httpd_app`, `type: front-end`.
- **Pod status**: All 4 pods in `Running` state.
- **Efficient YAML**: Clear and concise manifest.

**⚠️ Critical Configuration Details:**
- **ReplicaSet Name**: Must be `httpd-replicaset`.
- **Image**: Must be `httpd:latest`.
- **Replicas**: Must be 4.
- **Labels**: Must include `app: httpd_app`, `type: front-end`, `tier: frontend`.

**🔒 Kubernetes Benefits:**
- **High Availability**: Multiple replicas ensure redundancy.
- **Scalability**: ReplicaSet maintains desired pod count.
- **Manageability**: Labels enable service integration.

---

## ⚠️ Important Production Notes

🔧 **ReplicaSet**: Ensures high availability with multiple pod instances.

🔐 **Security**: Use specific image tags in production.

📊 **Monitoring**: Monitor pod health and resource usage.

🛡️ **Scalability**: Consider Deployments for rolling updates in production.

---