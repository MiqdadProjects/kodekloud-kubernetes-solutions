**🌟 Task 13 - Create Nginx ReplicationController**

**📌 Task Description**

The Nautilus DevOps team is deploying applications requiring high availability using ReplicationControllers to manage pod replicas. A team member has been tasked with creating a ReplicationController for nginx.

**Requirements:**
- Create a ReplicationController named `nginx-replicationcontroller` using `nginx:latest` image.
- Set labels: `app: nginx_app`, `type: front-end`.
- Name the container `nginx-container`.
- Ensure 3 replicas.
- Ensure all pods are running.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like ReplicationController names or labels may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy a ReplicationController with 3 nginx pods.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create ReplicationController Manifest

```bash
vi nginx-replicationcontroller.yaml
```

**Purpose**: Define the ReplicationController with 3 replicas.

**YAML Content**:
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-replicationcontroller
  labels:
    app: nginx_app
    type: front-end
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: ReplicationController`: Defines a ReplicationController resource.
- `metadata.labels`: Sets `app: nginx_app`, `type: front-end`.
- `spec.replicas`: Ensures 3 pod instances.
- `spec.selector`: Matches pods with `app: nginx` label.
- `spec.template`: Defines pod template with `nginx:latest` image.
- `ports`: Exposes port 80 for nginx.
- `imagePullPolicy`: Defaults to `IfNotPresent`.

---

## 🔹 Step 3: Apply the ReplicationController Manifest

```bash
kubectl apply -f nginx-replicationcontroller.yaml
```

**Purpose**: Deploy the ReplicationController.

**Expected Output**:
```
replicationcontroller/nginx-replicationcontroller created
```

---

## 🔹 Step 4: Verify ReplicationController

```bash
kubectl get rc
```

**Purpose**: Confirm the ReplicationController is created with 3 replicas.

**Expected Output**:
```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-replicationcontroller   3         3         3       10s
```

---

## 🔹 Step 5: Verify Pods

```bash
kubectl get pods -l app=nginx
```

**Purpose**: Ensure 3 pods are running.

**Expected Output**:
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-replicationcontroller-xxx   1/1     Running   0          10s
nginx-replicationcontroller-xxx   1/1     Running   0          10s
nginx-replicationcontroller-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 6: Verify ReplicationController Details

```bash
kubectl describe rc nginx-replicationcontroller
```

**Purpose**: Check ReplicationController configuration and pod status.

**Expected Output (partial)**:
```
Name:         nginx-replicationcontroller
Selector:     app=nginx
Labels:       app=nginx_app
              type=front-end
Replicas:     3 current / 3 desired
Containers:
  nginx-container:
    Image:        nginx:latest
```

---

## 📋 Quick Command Reference

```bash
# Create and apply ReplicationController manifest
vi nginx-replicationcontroller.yaml
# (Paste YAML content)
kubectl apply -f nginx-replicationcontroller.yaml

# Verify ReplicationController and pods
kubectl get rc
kubectl get pods -l app=nginx
kubectl describe rc nginx-replicationcontroller
```

---

## 💡 Additional Tips

- **Task Variability**: ReplicationController names or labels may differ; adjust YAML as needed.
- **Replicas**: Ensure cluster has enough resources for 3 pods.
- **Labels**: Match `selector` and `template` labels (`app: nginx`).
- **Dry Run**: Test with `kubectl apply -f nginx-replicationcontroller.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Fewer Pods Than Desired**
**Symptoms**: `kubectl get rc` shows fewer than 3 pods.
**Solution**: Check cluster resources.
```bash
kubectl describe rc nginx-replicationcontroller
# Look for resource shortage events
```

### **Issue 2: Label Mismatch**
**Symptoms**: Pods not managed by ReplicationController.
**Solution**: Verify `selector` and `template` labels.
```bash
kubectl get pods -l app=nginx
# Fix labels if mismatched
```

### **Issue 3: Image Pull Failure**
**Symptoms**: Pods in `ErrImagePull` state.
**Solution**: Verify image.
```bash
kubectl describe pod nginx-replicationcontroller-xxx
# Ensure image: nginx:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **ensuring 3 replicas** with correct labeling and container configuration for high availability.

**💡 Solution Approach:**
1. **ReplicationController Configuration**: Defined 3 replicas with `nginx:latest`.
2. **Label Consistency**: Used `app: nginx` for selector and pod template.
3. **Verification**: Confirmed 3 pods running with correct labels.
4. **Best Practices**: Included port 80 and minimal YAML.

**🎯 Key Success Factors:**
- **Replica count**: Set to 3 as required.
- **Label accuracy**: `app: nginx_app`, `type: front-end`.
- **Pod status**: All 3 pods in `Running` state.
- **Efficient YAML**: Clear and concise manifest.

**⚠️ Critical Configuration Details:**
- **ReplicationController Name**: Must be `nginx-replicationcontroller`.
- **Image**: Must be `nginx:latest`.
- **Replicas**: Must be 3.
- **Labels**: Must include `app: nginx_app`, `type: front-end`, `app: nginx`.

**🔒 Kubernetes Benefits:**
- **High Availability**: Multiple replicas ensure redundancy.
- **Scalability**: ReplicationController maintains desired pod count.
- **Manageability**: Labels enable service integration.

---

## ⚠️ Important Production Notes

🔧 **ReplicationController**: Ensures high availability with multiple pod instances.

🔐 **Security**: Use specific image tags in production.

📊 **Monitoring**: Monitor pod health and resource usage.

🛡️ **Scalability**: Consider Deployments for rolling updates in production.

---
