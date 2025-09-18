**🌟 Task 5 - Perform Rolling Update**

**📌 Task Description**

The Nautilus application development team has released a new version (nginx:1.18) for an application running on Kubernetes. The DevOps team needs to perform a rolling update to deploy this update without downtime.

**Requirements:**
- Update the `nginx-deployment` deployment to use the `nginx:1.18` image.
- Ensure all ports are accessible post-update.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like deployment names or ports may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Perform a rolling update to deploy the new nginx image.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Verify Existing Deployment

```bash
kubectl get deployment nginx-deployment
```

**Purpose**: Confirm the deployment exists and check its current state.

**Expected Output**:
```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment  1/1     1            1           1h
```

---

## 🔹 Step 3: Edit Deployment

```bash
kubectl edit deployment nginx-deployment
```

**Purpose**: Update the deployment to use the `nginx:1.18` image.

**YAML Change**:
Find the container section and update:
```yaml
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:1.18
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kubectl edit`: Opens the deployment YAML in the default editor (vi).
- `image: nginx:1.18`: Updates the container image to the specified version.

---

## 🔹 Step 4: Monitor Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

**Purpose**: Track the progress of the rolling update.

**Expected Output**:
```
deployment "nginx-deployment" successfully rolled out
```

---

## 🔹 Step 5: Verify Updated Pods

```bash
kubectl get pods -l app=nginx
```

**Purpose**: Confirm new pods are running with the updated image.

**Expected Output**:
```
NAME                             READY   STATUS    RESTARTS   AGE
nginx-deployment-xxx-xxx   1/1     Running   0          10s
```

**Verify Image**:
```bash
kubectl describe deployment nginx-deployment
```

**Expected Output (partial)**:
```
Containers:
  nginx:
    Image:        nginx:1.18
    State:        Running
```

---

## 🔹 Step 6: Verify Service Accessibility

```bash
kubectl get svc
curl http://<node-ip>:<node-port>
```

**Purpose**: Ensure the application is accessible post-update (assumes a service exists).

**Expected Output**:
```
Welcome to nginx!
```

---

## 📋 Quick Command Reference

```bash
# Verify deployment
kubectl get deployment nginx-deployment

# Update image
kubectl edit deployment nginx-deployment
# (Change image to nginx:1.18)

# Monitor and verify
kubectl rollout status deployment/nginx-deployment
kubectl get pods -l app=nginx
kubectl describe deployment nginx-deployment
```

---

## 💡 Additional Tips

- **Task Variability**: Deployment names or image versions may differ; verify existing configuration.
- **Zero Downtime**: Rolling updates ensure no service interruption.
- **Rollback Option**: Use `kubectl rollout undo` if the update fails.
- **Dry Run**: Test changes with `kubectl edit --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Update Stuck**
**Symptoms**: `kubectl rollout status` shows update not completing.
**Solution**: Check pod events.
```bash
kubectl describe pod nginx-deployment-xxx-xxx
# Look for image pull or resource issues
```

### **Issue 2: Image Pull Failure**
**Symptoms**: Pods in `ErrImagePull` state.
**Solution**: Verify image version.
```bash
kubectl describe deployment nginx-deployment
# Ensure image: nginx:1.18
```

### **Issue 3: Service Inaccessible**
**Symptoms**: `curl` fails post-update.
**Solution**: Verify service configuration.
```bash
kubectl describe svc nginx-service
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **performing a rolling update** without downtime while ensuring the correct image version is applied.

**💡 Solution Approach:**
1. **Image Update**: Used `kubectl edit` to update the image to `nginx:1.18`.
2. **Monitoring**: Tracked update progress with `kubectl rollout status`.
3. **Verification**: Confirmed new pods and service accessibility.
4. **Best Practices**: Ensured minimal disruption with rolling update strategy.

**🎯 Key Success Factors:**
- **Correct image**: Updated to `nginx:1.18`.
- **Zero downtime**: Rolling update maintained availability.
- **Verification**: Confirmed pod status and service access.
- **Efficient update**: Used `kubectl edit` for quick changes.

**⚠️ Critical Configuration Details:**
- **Deployment Name**: Must be `nginx-deployment`.
- **Image**: Must be `nginx:1.18`.
- **Service**: Assumes an existing service for accessibility.

**🔒 Kubernetes Benefits:**
- **Zero Downtime**: Rolling updates ensure continuous availability.
- **Reliability**: Automatic pod replacement.
- **Scalability**: Supports future replica scaling.

---

## ⚠️ Important Production Notes

🔧 **Rolling Updates**: Minimizes downtime during application updates.

🔐 **Security**: Specify exact image versions in production.

📊 **Monitoring**: Monitor rollout progress and pod health.

🛡️ **Rollback**: Keep previous images available for quick rollback.

---
