# 🌟 Task 10 - Fix Redis Deployment on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team deployed a Redis application on a Kubernetes cluster last week, which was functioning correctly until a team member made changes that caused the application to go down. The deployment named `redis-deployment` has pods that are not in a running state. The task is to investigate and fix the issue as soon as possible.

### Requirements:
- Fix the `redis-deployment` to ensure all pods are in a `Running` state.
- Address configuration issues, such as incorrect image names or ConfigMap references.
- Verify the deployment is running correctly with 1 replica.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like names or configurations may vary slightly, but the core challenge remains the same.

👉 **Your task**: Investigate and fix the Redis deployment to restore application functionality.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

## 🔹 Step 2: Investigate the Deployment Issue

```bash
kubectl describe deployment redis-deployment
```

**Purpose**: Inspect the deployment to identify issues.

**Key Observation**:
- **Incorrect Image**: The deployment uses `redis:alpin` (typo for `redis:alpine`).
- **Output Example**:
  ```
  Image:      redis:alpin
  ```

**Verification**: Confirm the image name is incorrect.

---

## 🔹 Step 3: Investigate Pod Issues

```bash
kubectl describe pod redis-deployment-<pod-hash>
```

**Purpose**: Check pod events to identify why pods are not running.

**Key Observation**:
- **Error**: Pods are failing due to a `FailedMount` issue.
- **Details**:
  ```
  Warning  FailedMount  2m2s (x10 over 6m12s)  kubelet  MountVolume.SetUp failed for volume "config" : configmap "redis-cofig" not found
  ```
- **Issue**: The deployment references a ConfigMap named `redis-cofig` (typo), but the actual ConfigMap is `redis-config`.

**Verification**: Confirm the ConfigMap name mismatch.

---

## 🔹 Step 4: Verify Available ConfigMaps

```bash
kubectl get configmaps
```

**Purpose**: List available ConfigMaps to confirm the correct name.

**Expected Output**:
```
NAME               DATA   AGE
kube-root-ca.crt   1      18m
redis-config       2      8m11s
```

**Verification**: Confirm the ConfigMap `redis-config` exists, and there is no `redis-cofig`.

---

## 🔹 Step 5: Fix the Deployment Configuration

```bash
kubectl edit deployment redis-deployment
```

**Purpose**: Correct the image name and ConfigMap reference in the deployment.

**Changes to Make**:
1. **Fix Image Name**:
   - Change `redis:alpin` to `redis:alpine`.
2. **Fix ConfigMap Name**:
   - Change `redis-cofig` to `redis-config` in the volume configuration.

**Sample YAML (Relevant Sections)**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: config
          mountPath: /redis-config
      volumes:
      - name: config
        configMap:
          name: redis-config
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `image: redis:alpine`: Corrects the image name to use the proper Redis Alpine image.
- `configMap.name: redis-config`: Corrects the ConfigMap name to match the existing `redis-config`.
- `replicas: 1`: Ensures one replica (based on provided solution output).

---

## 🔹 Step 6: Apply the Changes

**Note**: The `kubectl edit` command automatically applies changes. Alternatively, use a YAML file:

```bash
vi redis-deploy.yaml
```

**Paste the corrected YAML** (as shown above), then apply:

```bash
kubectl apply -f redis-deploy.yaml
```

**Expected Output**:
```
deployment.apps/redis-deployment configured
```

---

## 🔹 Step 7: Verify Deployment

```bash
kubectl get deployment
```

**Purpose**: Confirm the deployment is ready with 1 replica.

**Expected Output**:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           14m
```

**Verification**: Ensure the deployment shows `1/1` ready pods.

---

## 🔹 Step 8: Verify Pods

```bash
kubectl get pods
```

**Purpose**: Ensure all pods are running.

**Expected Output**:
```
NAME                              READY   STATUS    RESTARTS   AGE
redis-deployment-<pod-hash>       1/1     Running   0          2m
```

**Verification**: Confirm the pod is in `Running` state.

---

## 🔹 Step 9: Verify Image in Deployment

```bash
kubectl describe deployment redis-deployment | grep -i image
```

**Purpose**: Confirm the correct image is used.

**Expected Output**:
```
Image:      redis:alpine
```

**Verification**: Ensure the image is `redis:alpine`.

---

## 📋 Quick Command Reference

```bash
# Investigate issues
kubectl describe deployment redis-deployment
kubectl describe pod redis-deployment-<pod-hash>
kubectl get configmaps

# Fix deployment
kubectl edit deployment redis-deployment
# OR
vi redis-deploy.yaml
kubectl apply -f redis-deploy.yaml

# Verify resources
kubectl get deployment
kubectl get pods
kubectl describe deployment redis-deployment | grep -i image
```

---

## 💡 Additional Tips

- **Task Variability**: ConfigMap names or image tags may differ; always verify with `kubectl get configmaps`.
- **Pod Startup**: Redis pods may take time to initialize; monitor pod status with `kubectl get pods -w`.
- **Dry Run**: Test changes with `kubectl apply -f redis-deploy.yaml --dry-run=client`.
- **ConfigMap Verification**: Ensure the ConfigMap `redis-config` contains the expected Redis configuration.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pods Not Running**
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod redis-deployment-<pod-hash>
kubectl logs redis-deployment-<pod-hash>
```

### **Issue 2: ConfigMap Not Found**
**Symptoms**: `FailedMount` error for `redis-cofig`.
**Solution**: Verify ConfigMap name and correct it in the deployment.
```bash
kubectl get configmaps
kubectl edit deployment redis-deployment
# Ensure configMap.name: redis-config
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status due to `redis:alpin`.
**Solution**: Correct the image name to `redis:alpine`.
```bash
kubectl describe pod redis-deployment-<pod-hash>
kubectl edit deployment redis-deployment
# Ensure image: redis:alpine
```

### **Issue 4: Deployment Not Updating**
**Symptoms**: Changes not reflected after `kubectl edit`.
**Solution**: Force a redeployment or apply a YAML file.
```bash
kubectl rollout restart deployment redis-deployment
```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenge Encountered:

The primary challenge was **fixing a broken Redis deployment** caused by:
1. An incorrect image name (`redis:alpin` instead of `redis:alpine`).
2. A typo in the ConfigMap name (`redis-cofig` instead of `redis-config`).

### 💡 Solution Approach:
1. **Diagnosed Issues**: Used `kubectl describe` to identify the incorrect image and ConfigMap errors.
2. **Corrected Configuration**: Updated the deployment to use `redis:alpine` and `redis-config`.
3. **Verified Fix**: Ensured the deployment and pods were running with the correct configuration.
4. **Best Practices**: Used precise commands to minimize downtime and verified all resources.

### 🎯 Key Success Factors:
- **Correct Image**: `redis:alpine` ensures the correct Redis image.
- **Correct ConfigMap**: `redis-config` resolves the `FailedMount` issue.
- **Pod Status**: All pods in `Running` state.
- **Deployment Verification**: `1/1` ready replicas.

### ⚠️ Critical Configuration Details:
- **Deployment Name**: Must be `redis-deployment`.
- **Image**: Must be `redis:alpine`.
- **ConfigMap**: Must reference `redis-config`.
- **Replicas**: 1 (based on provided output).

### 🔒 Kubernetes Benefits:
- **Reliability**: Correct configurations ensure stable Redis deployment.
- **Manageability**: Deployments handle pod lifecycle and updates.
- **Debugging**: Kubernetes events and logs provide clear diagnostics.

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.

🔐 **Security**: Secure Redis with authentication and network policies.

📊 **Monitoring**: Monitor Redis pod health and performance metrics.

🛡️ **Scalability**: Adjust replicas or use Redis cluster for high availability.

---