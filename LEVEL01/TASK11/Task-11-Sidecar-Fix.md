**🌟 Task 11 - Fix Sidecar Pod**

**📌 Task Description**

A junior DevOps team member deployed a pod with a sidecar container, but it’s failing to run. The team needs to diagnose and fix the issue to ensure the pod is operational and the application is accessible.

**Requirements:**
- Fix the pod named `webserver` with container `httpd-container` using `httpd:latest` image.
- Fix the sidecar container named `sidecar-container` using `ubuntu:latest` image.
- Ensure the pod is in a running state and the application is accessible.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names or image tags may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Diagnose and fix the pod to ensure it runs correctly.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Diagnose the Pod Issue

```bash
kubectl describe pod webserver
```

**Purpose**: Identify the issue with the pod.

**Expected Output (partial)**:
```
Name:         webserver
Containers:
  httpd-container:
    Image:        httpd:latests
    State:        ErrImagePull
```

**Issue Identified**: The `httpd-container` image is incorrectly set to `httpd:latests` (typo).

---

## 🔹 Step 3: Fix the Image Name

```bash
kubectl set image pod/webserver httpd-container=httpd:latest
```

**Purpose**: Update the `httpd-container` image to `httpd:latest`.

**Expected Output**:
```
pod/webserver image updated
```

**Command Breakdown**:
- `kubectl set image`: Updates the container image in a pod.
- `pod/webserver`: Targets the `webserver` pod.
- `httpd-container=httpd:latest`: Sets the correct image for `httpd-container`.

**Alternative Approach** (if needed):
```bash
kubectl edit pod webserver
# Change image: httpd:latests to image: httpd:latest
# Save and exit
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
webserver   2/2     Running   0          10s
```

---

## 🔹 Step 5: Verify Container Images

```bash
kubectl describe pod webserver
```

**Purpose**: Check that both containers use the correct images.

**Expected Output (partial)**:
```
Containers:
  httpd-container:
    Image:        httpd:latest
    State:        Running
  sidecar-container:
    Image:        ubuntu:latest
    State:        Running
```

---

## 🔹 Step 6: Verify Application Accessibility

```bash
kubectl get svc
curl http://<node-ip>:<node-port>
```

**Purpose**: Ensure the application is accessible (assumes a service exists).

**Expected Output**:
```
Welcome to Apache!
```

---

## 📋 Quick Command Reference

```bash
# Diagnose issue
kubectl describe pod webserver

# Fix image
kubectl set image pod/webserver httpd-container=httpd:latest

# Verify pod and application
kubectl get pods
kubectl describe pod webserver
curl http://<node-ip>:<node-port>
```

---

## 💡 Additional Tips

- **Task Variability**: Pod names or image typos may differ; always check `kubectl describe`.
- **Sidecar Pattern**: Ensure sidecar container complements the main container (e.g., logging).
- **Service Check**: Verify service configuration if `curl` fails.
- **Dry Run**: Test image update with `kubectl set image --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Pod in `ErrImagePull` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod webserver
kubectl logs webserver -c httpd-container
kubectl logs webserver -c sidecar-container
```

### **Issue 2: Image Pull Failure**
**Symptoms**: `ErrImagePull` for `httpd-container`.
**Solution**: Verify image name.
```bash
kubectl describe pod webserver
# Ensure image: httpd:latest
```

### **Issue 3: Sidecar Container Failure**
**Symptoms**: `sidecar-container` not running.
**Solution**: Check sidecar configuration.
```bash
kubectl describe pod webserver
# Verify ubuntu:latest and command
```

### **Issue 4: Application Inaccessible**
**Symptoms**: `curl` fails.
**Solution**: Verify service and port configuration.
```bash
kubectl describe svc
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **diagnosing and fixing an incorrect image name** (`httpd:latests`) in a multi-container pod with a sidecar.

**💡 Solution Approach:**
1. **Diagnosis**: Used `kubectl describe` to identify the typo in `httpd-container` image.
2. **Fix**: Corrected the image to `httpd:latest` using `kubectl set image`.
3. **Verification**: Confirmed pod status and application accessibility.
4. **Best Practices**: Ensured both containers are running and service is accessible.

**🎯 Key Success Factors:**
- **Accurate diagnosis**: Identified typo via `kubectl describe`.
- **Quick fix**: Used `kubectl set image` for minimal disruption.
- **Verification**: Confirmed `Running` state and accessibility.
- **Sidecar awareness**: Ensured sidecar container is functional.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `webserver`.
- **Containers**: `httpd-container` (`httpd:latest`), `sidecar-container` (`ubuntu:latest`).
- **Service**: Assumes an existing service for accessibility.

**🔒 Kubernetes Benefits:**
- **Sidecar Pattern**: Enhances main container functionality.
- **Reliability**: Multi-container pods support complex workflows.
- **Quick Fixes**: `kubectl set image` allows rapid corrections.

---

## ⚠️ Important Production Notes

🔧 **Sidecar Containers**: Useful for logging or monitoring in production.

🔐 **Security**: Ensure sidecar images are minimal and secure.

📊 **Monitoring**: Monitor both containers’ resource usage.

🛡️ **Scalability**: Use Deployments for managing multi-container pods.

---
