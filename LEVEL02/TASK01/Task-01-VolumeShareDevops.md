**🌟 Task 1 - Create Pod with Shared Volume**

**📌 Task Description**

The Nautilus DevOps team is working on an application requiring multiple containers within a pod to share a volume for temporary data. They are developing a template to replicate this scenario.

**Requirements:**
- Create a pod named `volume-share-devops`.
- First container: Use `debian:latest` image, named `volume-container-devops-1`, run `sleep 2000`, mount volume `volume-share` at `/tmp/blog`.
- Second container: Use `debian:latest` image, named `volume-container-devops-2`, run `sleep 2000`, mount volume `volume-share` at `/tmp/apps`.
- Volume: Name `volume-share`, type `emptyDir` (no size limit or medium specified).
- Exec into `volume-container-devops-1`, create `blog.txt` with any content at `/tmp/blog`.
- Verify `blog.txt` exists at `/tmp/apps` in `volume-container-devops-2`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names or paths may differ in your environment.

👉 **Your task**: Deploy a pod with a shared volume and test file sharing.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Generate and Edit Pod Manifest

```bash
kubectl run volume-share-devops --image=debian:latest --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

**Purpose**: Create a pod manifest with shared volume configuration.

**YAML Content**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
spec:
  containers:
  - name: volume-container-devops-1
    image: debian:latest
    imagePullPolicy: IfNotPresent
    command: ["sleep", "2000"]
    volumeMounts:
    - mountPath: /tmp/blog
      name: volume-share
  - name: volume-container-devops-2
    image: debian:latest
    imagePullPolicy: IfNotPresent
    command: ["sleep", "2000"]
    volumeMounts:
    - mountPath: /tmp/apps
      name: volume-share
  volumes:
  - name: volume-share
    emptyDir: {}
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Pod`: Defines a pod resource.
- `metadata.name`: Sets pod name to `volume-share-devops`.
- `spec.containers`: Defines two containers with `debian:latest`.
- `command: ["sleep", "2000"]`: Keeps containers running.
- `volumeMounts`: Mounts `volume-share` at `/tmp/blog` and `/tmp/apps`.
- `volumes`: Defines `emptyDir` volume named `volume-share`.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

**Note**: Removed `sizeLimit: 200Mi` and `medium: Memory` from the provided solution, as they were not specified in the requirements.

---

## 🔹 Step 3: Apply the Pod Manifest

```bash
kubectl apply -f pod.yaml
```

**Purpose**: Deploy the pod with shared volume.

**Expected Output**:
```
pod/volume-share-devops created
```

---

## 🔹 Step 4: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm the pod is running.

**Expected Output**:
```
NAME                 READY   STATUS    RESTARTS   AGE
volume-share-devops  2/2     Running   0          10s
```

---

## 🔹 Step 5: Create File in First Container

```bash
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- sh
echo "This is a test file for the shared volume." > /tmp/blog/blog.txt
ls /tmp/blog
cat /tmp/blog/blog.txt
exit
```

**Purpose**: Create and verify `blog.txt` in the first container.

**Expected Output**:
```
blog.txt
This is a test file for the shared volume.
```

---

## 🔹 Step 6: Verify File in Second Container

```bash
kubectl exec -it volume-share-devops -c volume-container-devops-2 -- sh
ls /tmp/apps
cat /tmp/apps/blog.txt
exit
```

**Purpose**: Confirm `blog.txt` exists in the second container’s mount path.

**Expected Output**:
```
blog.txt
This is a test file for the shared volume.
```

---

## 📋 Quick Command Reference

```bash
# Create and apply pod manifest
kubectl run volume-share-devops --image=debian:latest --dry-run=client -o yaml > pod.yaml
vi pod.yaml
# (Paste YAML content)
kubectl apply -f pod.yaml

# Verify pod and test file sharing
kubectl get pods
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- sh -c 'echo "This is a test file for the shared volume." > /tmp/blog/blog.txt; ls /tmp/blog; cat /tmp/blog/blog.txt'
kubectl exec -it volume-share-devops -c volume-container-devops-2 -- sh -c 'ls /tmp/apps; cat /tmp/apps/blog.txt'
```

---

## 💡 Additional Tips

- **Task Variability**: Pod names or mount paths may differ; adjust YAML as needed.
- **Shared Volume**: `emptyDir` enables data sharing between containers.
- **Persistence**: `emptyDir` is ephemeral; use PersistentVolumes for production.
- **Dry Run**: Test with `kubectl apply -f pod.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod volume-share-devops
kubectl logs volume-share-devops -c volume-container-devops-1
kubectl logs volume-share-devops -c volume-container-devops-2
```

### **Issue 2: File Not Shared**
**Symptoms**: `blog.txt` not visible in second container.
**Solution**: Verify volume mount paths and `emptyDir` configuration.
```bash
kubectl describe pod volume-share-devops
# Ensure volumeMounts and volumes are correct
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod volume-share-devops
# Ensure image: debian:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **configuring a shared volume** between two containers and verifying file sharing across different mount paths.

**💡 Solution Approach:**
1. **Pod Configuration**: Defined a pod with two `debian:latest` containers.
2. **Shared Volume**: Used `emptyDir` volume mounted at `/tmp/blog` and `/tmp/apps`.
3. **File Testing**: Created `blog.txt` in the first container and verified in the second.
4. **Verification**: Confirmed pod status and file sharing with `kubectl exec`.

**🎯 Key Success Factors:**
- **Correct volume setup**: `emptyDir` shared between containers.
- **Accurate mount paths**: `/tmp/blog` and `/tmp/apps`.
- **File sharing**: `blog.txt` accessible in both containers.
- **Minimal configuration**: Used `sleep 2000` to keep containers running.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `volume-share-devops`.
- **Container Names**: `volume-container-devops-1`, `volume-container-devops-2`.
- **Image**: Must be `debian:latest`.
- **Volume**: `volume-share` of type `emptyDir`.
- **Mount Paths**: `/tmp/blog` and `/tmp/apps`.
- **File**: `blog.txt` created in `/tmp/blog`.

**🔒 Kubernetes Benefits:**
- **Shared Volumes**: `emptyDir` enables data sharing within a pod.
- **Flexibility**: Multiple containers can access the same volume.
- **Simplicity**: Minimal setup for temporary data storage.

---

## ⚠️ Important Production Notes

🔧 **Volume Management**: Use PersistentVolumes for durable storage in production.

🔐 **Security**: Restrict container permissions for file access.

📊 **Monitoring**: Monitor pod resource usage and volume growth.

🛡️ **Scalability**: Consider multi-container patterns for complex apps.

---
