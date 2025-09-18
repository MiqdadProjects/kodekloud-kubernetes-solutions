**🌟 Task 6 - Rollback Nginx Deployment**

**📌 Task Description**

The Nautilus DevOps team rolled out a new release, but a customer reported a bug. The team needs to rollback the `nginx-deployment` to its previous revision to restore stability.

**Requirements:**
- Roll back the `nginx-deployment` deployment to the previous revision.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like deployment names may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Revert the nginx deployment to its previous state.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Check Deployment History

```bash
kubectl rollout history deployment/nginx-deployment
```

**Purpose**: View the revision history to confirm rollback feasibility.

**Expected Output**:
```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

**Command Breakdown**:
- `kubectl rollout history`: Displays revision history for the deployment.
- `REVISION`: Lists available revisions (e.g., 1, 2).

---

## 🔹 Step 3: Perform Rollback

```bash
kubectl rollout undo deployment/nginx-deployment
```

**Purpose**: Revert the deployment to the previous revision.

**Expected Output**:
```
deployment.apps/nginx-deployment rolled back
```

**Command Breakdown**:
- `kubectl rollout undo`: Reverts to the previous revision (e.g., from 2 to 1).
- `deployment/nginx-deployment`: Specifies the target deployment.

---

## 🔹 Step 4: Monitor Rollback

```bash
kubectl rollout status deployment/nginx-deployment
```

**Purpose**: Confirm the rollback completed successfully.

**Expected Output**:
```
deployment "nginx-deployment" successfully rolled out
```

---

## 🔹 Step 5: Verify Pods

```bash
kubectl get pods -l app=nginx
```

**Purpose**: Ensure new pods are running with the previous image.

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
    Image:        nginx:<previous-version>
    State:        Running
```

---

## 🔹 Step 6: Verify Service Accessibility

```bash
kubectl get svc
curl http://<node-ip>:<node-port>
```

**Purpose**: Confirm the application is accessible post-rollback.

**Expected Output**:
```
Welcome to nginx!
```

---

## 📋 Quick Command Reference

```bash
# Check history and rollback
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment

# Verify rollback
kubectl rollout status deployment/nginx-deployment
kubectl get pods -l app=nginx
kubectl describe deployment nginx-deployment
```

---

## 💡 Additional Tips

- **Task Variability**: Deployment names or previous image versions may differ; verify history.
- **Rollback Safety**: Always check revision history before undoing.
- **Monitoring**: Monitor pod health during rollback.
- **Documentation**: Record `CHANGE-CAUSE` in production with `--record`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Rollback Fails**
**Symptoms**: `kubectl rollout undo` fails or no previous revision.
**Solution**: Check revision history.
```bash
kubectl rollout history deployment/nginx-deployment
# Ensure multiple revisions exist
```

### **Issue 2: Pods Not Running**
**Symptoms**: Pods in `CrashLoopBackOff` post-rollback.
**Solution**: Check pod logs.
```bash
kubectl logs nginx-deployment-xxx-xxx
```

### **Issue 3: Service Inaccessible**
**Symptoms**: `curl` fails post-rollback.
**Solution**: Verify service configuration.
```bash
kubectl describe svc nginx-service
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **reverting a deployment** to a stable previous revision without disrupting service.

**💡 Solution Approach:**
1. **History Check**: Used `kubectl rollout history` to confirm revisions.
2. **Rollback Execution**: Applied `kubectl rollout undo` to revert.
3. **Verification**: Confirmed pod status and service accessibility.
4. **Best Practices**: Monitored rollout status to ensure success.

**🎯 Key Success Factors:**
- **Accurate rollback**: Reverted to previous revision.
- **No downtime**: Rollback maintained availability.
- **Verification**: Confirmed previous image and pod status.
- **Simple command**: Used `kubectl rollout undo` for efficiency.

**⚠️ Critical Configuration Details:**
- **Deployment Name**: Must be `nginx-deployment`.
- **Revisions**: Requires at least two revisions for rollback.
- **Service**: Assumes an existing service for accessibility.

**🔒 Kubernetes Benefits:**
- **Reliability**: Rollbacks restore stable configurations.
- **Zero Downtime**: Rolling strategy minimizes disruption.
- **Version Control**: Deployment history tracks changes.

---

## ⚠️ Important Production Notes

🔧 **Rollback Strategy**: Essential for quick recovery from faulty updates.

🔐 **Security**: Verify previous image security before rollback.

📊 **Monitoring**: Track rollback progress and pod health.

🛡️ **Documentation**: Use `--record` to log change causes.

---