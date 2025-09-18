**🌟 Task 2 - Create Webserver Pod with Sidecar**

**📌 Task Description**

The Nautilus DevOps team needs a pod with a webserver and a sidecar container to monitor logs using a shared volume.

**Requirements:**
- Create a pod named `webserver`.
- Create an `emptyDir` volume named `shared-logs`.
- First container: Use `nginx:latest` image, named `nginx-container`, mount `shared-logs` at `/var/log/nginx`.
- Second container: Use `ubuntu:latest` image, named `sidecar-container`, run command `sh -c "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"`, mount `shared-logs` at `/var/log/nginx`.
- Ensure all containers are running.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names or paths may differ in your environment.

👉 **Your task**: Deploy a pod with a shared logging volume.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Pod Manifest

```bash
vi webserver.yaml
```

**Purpose**: Define the pod with shared volume and sidecar container.

**YAML Content**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}
  containers:
  - name: nginx-container
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: sidecar-container
    image: ubuntu:latest
    imagePullPolicy: IfNotPresent
    command: ["sh", "-c", "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Pod`: Defines a pod resource.
- `volumes`: Creates `emptyDir` volume `shared-logs`.
- `containers`: Defines `nginx-container` and `sidecar-container`.
- `volumeMounts`: Mounts `shared-logs` at `/var/log/nginx` for both containers.
- `command`: Runs logging loop in `sidecar-container`.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply the Pod Manifest

```bash
kubectl apply -f webserver.yaml
```

**Purpose**: Deploy the pod with shared volume.

**Expected Output**:
```
pod/webserver created
```

---

## 🔹 Step 4: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm both containers are running.

**Expected Output**:
```
NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          10s
```

---

## 🔹 Step 5: Verify Logs in Sidecar Container

```bash
kubectl logs webserver -c sidecar-container
```

**Purpose**: Check the sidecar container’s log output.

**Expected Output**:
```
<nginx access and error log entries>
```

---

## 📋 Quick Command Reference

```bash
# Create and apply pod manifest
vi webserver.yaml
# (Paste YAML content)
kubectl apply -f webserver.yaml

# Verify pod and logs
kubectl get pods
kubectl logs webserver -c sidecar-container
```

---

## 💡 Additional Tips

- **Task Variability**: Container names or mount paths may differ; adjust YAML as needed.
- **Sidecar Pattern**: Useful for logging or monitoring.
- **Volume Scope**: `emptyDir` is pod-scoped; use PersistentVolumes for persistence.
- **Dry Run**: Test with `kubectl apply -f webserver.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Containers in `CrashLoopBackOff` or `Pending`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod webserver
kubectl logs webserver -c nginx-container
kubectl logs webserver -c sidecar-container
```

### **Issue 2: Logs Not Accessible**
**Symptoms**: Sidecar container shows no logs.
**Solution**: Verify volume mount and command.
```bash
kubectl describe pod webserver
# Ensure volumeMounts and command are correct
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image names.
```bash
kubectl describe pod webserver
# Ensure images: nginx:latest, ubuntu:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **setting up a sidecar container** to monitor Nginx logs using a shared volume.

**💡 Solution Approach:**
1. **Pod Configuration**: Defined a pod with `nginx` and `ubuntu` containers.
2. **Shared Volume**: Used `emptyDir` to share logs at `/var/log/nginx`.
3. **Sidecar Command**: Configured logging loop in `sidecar-container`.
4. **Verification**: Confirmed pod status and log output.

**🎯 Key Success Factors:**
- **Correct volume setup**: `shared-logs` mounted correctly.
- **Sidecar functionality**: Logging loop outputs Nginx logs.
- **Pod status**: Both containers running.
- **Minimal configuration**: Efficient YAML setup.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `webserver`.
- **Volume**: `shared-logs` of type `emptyDir`.
- **Containers**: `nginx-container` (`nginx:latest`), `sidecar-container` (`ubuntu:latest`).
- **Mount Path**: `/var/log/nginx` for both containers.
- **Command**: Logging loop in `sidecar-container`.

**🔒 Kubernetes Benefits:**
- **Sidecar Pattern**: Enhances logging and monitoring.
- **Shared Volumes**: Simplifies data sharing within pods.
- **Flexibility**: Supports multi-container workflows.

---

## ⚠️ Important Production Notes

🔧 **Logging**: Use centralized logging (e.g., Fluentd, ELK) in production.

🔐 **Security**: Restrict container permissions for log access.

📊 **Monitoring**: Monitor log volume growth and container health.

🛡️ **Scalability**: Consider Deployments for scalable web servers.

---