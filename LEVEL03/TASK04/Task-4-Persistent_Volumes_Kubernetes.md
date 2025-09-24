# 🌟 Task 4 - Deploy Web Application with Persistent Volume on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team is developing a Kubernetes template to deploy a web application using a persistent volume to store the application code. The template must meet specific requirements for persistent storage, pod configuration, and service exposure to ensure the application is accessible and reliable.

### Requirements:
- Create a PersistentVolume named **`pv-datacenter`** with:
  - Storage class set to `manual`.
  - Capacity of `4Gi`.
  - Access mode set to `ReadWriteOnce`.
  - Volume type set to `hostPath` with path `/mnt/finance` (pre-existing directory).
- Create a PersistentVolumeClaim named **`pvc-datacenter`** with:
  - Storage class set to `manual`.
  - Request `2Gi` of storage.
  - Access mode set to `ReadWriteOnce`.
- Create a pod named **`pod-datacenter`** with:
  - A container named **`container-datacenter`** using the `httpd:latest` image.
  - Mount the persistent volume (via claim `pvc-datacenter`) at the Apache document root (`/usr/local/apache2/htdocs`).
- Create a NodePort service named **`web-datacenter`** to expose the web server on **NodePort 30008**.
- Ensure the `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Deploy a web application on the Kubernetes cluster with a persistent volume to store application code, ensuring accessibility via NodePort 30008.

💡 **Note**: The `/mnt/finance` directory is pre-existing on the host, and direct access is not required for this task.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- PersistentVolume for storage
- PersistentVolumeClaim to request storage
- Pod running an Apache web server
- NodePort service to expose the web application

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **PersistentVolume**: Provides storage backed by a host path
- **PersistentVolumeClaim**: Requests storage from the PersistentVolume
- **Pod**: Runs the Apache web server with a mounted volume
- **Service**: Exposes the web server externally via NodePort
- **Labels**: Ensures proper pod selection for the service

### 🎯 Implementation Strategy
1. Create a PersistentVolume with the specified storage and host path
2. Create a PersistentVolumeClaim to request storage
3. Deploy a pod with the Apache container and mount the persistent volume
4. Expose the pod via a NodePort service on port 30008
5. Verify the resources and application accessibility

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Configuration File
Create a file named `datacenter-deployment.yaml`:

```yaml
# datacenter-deployment.yaml
# Define PersistentVolume for storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-datacenter
spec:
  storageClassName: "manual"
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/finance
---
# Define PersistentVolumeClaim to request storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-datacenter
spec:
  storageClassName: "manual"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
# Define Pod with Apache web server
apiVersion: v1
kind: Pod
metadata:
  name: pod-datacenter
  labels:
    app.kubernetes.io/name: pod-datacenter
spec:
  containers:
    - name: container-datacenter
      image: httpd:latest
      imagePullPolicy: IfNotPresent
      volumeMounts:
        - mountPath: /usr/local/apache2/htdocs
          name: shared-logs
  volumes:
    - name: shared-logs
      persistentVolumeClaim:
        claimName: pvc-datacenter
  restartPolicy: Always
---
# Define NodePort service to expose the web server
apiVersion: v1
kind: Service
metadata:
  name: web-datacenter
spec:
  selector:
    app.kubernetes.io/name: pod-datacenter
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
  type: NodePort
```

**Configuration Breakdown:**
- **PersistentVolume**:
  - `name: pv-datacenter` - Specifies the PersistentVolume name
  - `storageClassName: manual` - Uses manual storage class
  - `capacity.storage: 4Gi` - Sets storage capacity
  - `accessModes: ReadWriteOnce` - Allows one node to read/write
  - `hostPath.path: /mnt/finance` - Uses pre-existing host directory
- **PersistentVolumeClaim**:
  - `name: pvc-datacenter` - Specifies the claim name
  - `storageClassName: manual` - Matches the PersistentVolume
  - `resources.requests.storage: 2Gi` - Requests 2Gi of storage
  - `accessModes: ReadWriteOnce` - Matches the PersistentVolume
- **Pod**:
  - `name: pod-datacenter` - Specifies the pod name
  - `labels: app.kubernetes.io/name: pod-datacenter` - Labels for service selection
  - `containers.image: httpd:latest` - Uses Apache image
  - `volumeMounts.mountPath: /usr/local/apache2/htdocs` - Mounts volume at Apache document root
  - `volumes.persistentVolumeClaim.claimName: pvc-datacenter` - Links to PVC
- **Service**:
  - `name: web-datacenter` - Specifies the service name
  - `type: NodePort` - Exposes the service externally
  - `nodePort: 30008` - Sets the external port
  - `selector: app.kubernetes.io/name: pod-datacenter` - Links to the pod

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configuration
```bash
kubectl apply -f datacenter-deployment.yaml
```

**Purpose:** Deploy the PersistentVolume, PersistentVolumeClaim, pod, and service to the Kubernetes cluster.

**Expected Output:**
```
persistentvolume/pv-datacenter created
persistentvolumeclaim/pvc-datacenter created
pod/pod-datacenter created
service/web-datacenter created
```

---

### Step 4: Verify PersistentVolume and Claim
```bash
kubectl get pv pv-datacenter
```

**Expected Output:**
```
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   AGE
pv-datacenter   4Gi        RWO            Retain           Bound    default/pvc-datacenter    manual         2m
```

**Verification:** Confirm the PersistentVolume is bound to `pvc-datacenter`.

```bash
kubectl get pvc pvc-datacenter
```

**Expected Output:**
```
NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-datacenter   Bound    pv-datacenter   4Gi        RWO            manual         2m
```

**Verification:** Confirm the PersistentVolumeClaim is bound.

---

### Step 5: Verify Pod
```bash
kubectl get pod pod-datacenter
```

**Purpose:** Ensure the pod is in a `Running` state.

**Expected Output:**
```
NAME             READY   STATUS    RESTARTS   AGE
pod-datacenter   1/1     Running   0          2m
```

**Verification:** Confirm the pod is running.

---

### Step 6: Verify Service
```bash
kubectl get svc web-datacenter
```

**Purpose:** Check the NodePort configuration.

**Expected Output:**
```
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-datacenter   NodePort   10.96.123.456   <none>        80:30008/TCP   2m
```

**Verification:** Confirm the service uses NodePort `30008`.

---

### Step 7: Verify Application Accessibility
**Access the Application**:
- Click on the **App** or **Website** button in the lab interface.
- Alternatively, use the cluster's external IP or Node IP with port `30008`:
  ```bash
  curl http://<node-ip>:30008
  ```

**Expected Output:**
```
<html><body><h1>It works!</h1></body></html>
```

**Verification:** Confirm the Apache default page is accessible, indicating the web server is running with the persistent volume mounted.

---

## 🔍 Code Analysis

### Complete Configuration (`datacenter-deployment.yaml`)
```yaml
# Define PersistentVolume for storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-datacenter
spec:
  storageClassName: "manual"
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/finance
---
# Define PersistentVolumeClaim to request storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-datacenter
spec:
  storageClassName: "manual"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
# Define Pod with Apache web server
apiVersion: v1
kind: Pod
metadata:
  name: pod-datacenter
  labels:
    app.kubernetes.io/name: pod-datacenter
spec:
  containers:
    - name: container-datacenter
      image: httpd:latest
      imagePullPolicy: IfNotPresent
      volumeMounts:
        - mountPath: /usr/local/apache2/htdocs
          name: shared-logs
  volumes:
    - name: shared-logs
      persistentVolumeClaim:
        claimName: pvc-datacenter
  restartPolicy: Always
---
# Define NodePort service to expose the web server
apiVersion: v1
kind: Service
metadata:
  name: web-datacenter
spec:
  selector:
    app.kubernetes.io/name: pod-datacenter
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
  type: NodePort
```

### PersistentVolume Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | pv-datacenter | Name of the PersistentVolume |
| **storageClassName** | manual | Manual storage class |
| **capacity.storage** | 4Gi | Storage capacity |
| **accessModes** | ReadWriteOnce | Single node read/write access |
| **hostPath.path** | /mnt/finance | Host directory for storage |

### PersistentVolumeClaim Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | pvc-datacenter | Name of the claim |
| **storageClassName** | manual | Matches PersistentVolume |
| **resources.requests.storage** | 2Gi | Requested storage |
| **accessModes** | ReadWriteOnce | Matches PersistentVolume |

### Pod Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | pod-datacenter | Name of the pod |
| **labels** | app.kubernetes.io/name: pod-datacenter | Label for service selection |
| **containers.image** | httpd:latest | Apache web server image |
| **volumeMounts.mountPath** | /usr/local/apache2/htdocs | Apache document root |
| **volumes.persistentVolumeClaim** | pvc-datacenter | Links to PVC |

### Service Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | web-datacenter | Name of the service |
| **type** | NodePort | Exposes service externally |
| **nodePort** | 30008 | External port |
| **selector** | app.kubernetes.io/name: pod-datacenter | Links to pod |

---

## ✅ Verification Steps

### Step 1: Verify PersistentVolume
```bash
kubectl get pv pv-datacenter
```

**Expected Output:**
```
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   AGE
pv-datacenter   4Gi        RWO            Retain           Bound    default/pvc-datacenter    manual         2m
```

### Step 2: Verify PersistentVolumeClaim
```bash
kubectl get pvc pvc-datacenter
```

**Expected Output:**
```
NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-datacenter   Bound    pv-datacenter   4Gi        RWO            manual         2m
```

### Step 3: Verify Pod
```bash
kubectl get pod pod-datacenter
```

**Expected Output:**
```
NAME             READY   STATUS    RESTARTS   AGE
pod-datacenter   1/1     Running   0          2m
```

### Step 4: Verify Service
```bash
kubectl get svc web-datacenter
```

**Expected Output:**
```
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-datacenter   NodePort   10.96.123.456   <none>        80:30008/TCP   2m
```

### Step 5: Test Application
```bash
curl http://<node-ip>:30008
```

**Expected Output:**
```
<html><body><h1>It works!</h1></body></html>
```

---

## 🧪 Testing

### Verify Volume Mount
```bash
kubectl exec pod-datacenter -c container-datacenter -- ls /usr/local/apache2/htdocs
```

**Purpose:** Confirm the persistent volume is mounted at the Apache document root.

### Verify Pod Status
```bash
kubectl describe pod pod-datacenter
```

**Purpose:** Check pod events for any issues with volume mounting or container startup.

### Verify Service Accessibility
```bash
kubectl get nodes -o wide
curl http://<node-ip>:30008
```

**Purpose:** Confirm the web server is accessible via NodePort.

---

## 📚 Quick Command Reference

```bash
# Apply configuration
kubectl apply -f datacenter-deployment.yaml

# Verify resources
kubectl get pv pv-datacenter
kubectl get pvc pvc-datacenter
kubectl get pod pod-datacenter
kubectl get svc web-datacenter

# Test application
curl http://<node-ip>:30008
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: PersistentVolume Not Bound
**Symptoms:** PVC status is `Pending` or not bound.
**Solution:** Verify PV and PVC configurations match.
```bash
kubectl describe pv pv-datacenter
kubectl describe pvc pvc-datacenter
```

### Issue 2: Pod Not Running
**Symptoms:** Pod in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod pod-datacenter
kubectl logs pod-datacenter -c container-datacenter
```

### Issue 3: Service Not Accessible
**Symptoms:** Unable to access application on NodePort 30008.
**Solution:** Verify service selector and node IP.
```bash
kubectl describe svc web-datacenter
kubectl get nodes -o wide
```

### Issue 4: Volume Mount Failure
**Symptoms:** Apache document root is empty or inaccessible.
**Solution:** Verify volume mount and PVC binding.
```bash
kubectl exec pod-datacenter -c container-datacenter -- ls /usr/local/apache2/htdocs
kubectl describe pod pod-datacenter
```

---

## 💡 Additional Tips

- **Pod Logs**: Check logs for Apache errors.
  ```bash
  kubectl logs pod-datacenter -c container-datacenter
  ```
- **Dry Run**: Test configuration before applying.
  ```bash
  kubectl apply -f datacenter-deployment.yaml --dry-run=client
  ```
- **Volume Verification**: Ensure the volume is mounted correctly.
  ```bash
  kubectl exec pod-datacenter -c container-datacenter -- df -h /usr/local/apache2/htdocs
  ```
- **NodePort Range**: Verify NodePort 30008 is within the cluster’s allowed range (30000-32767).

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **PersistentVolume Configuration**: Setting up `hostPath` with the correct path and storage class.
2. **PVC Binding**: Ensuring the PVC binds to the PV with matching storage class and access mode.
3. **Volume Mounting**: Correctly mounting the volume at the Apache document root.
4. **Service Exposure**: Configuring the NodePort service to select the pod correctly.

### 💡 Solution Approach:
1. **PersistentVolume**: Configured with `manual` storage class and `hostPath` at `/mnt/finance`.
2. **PersistentVolumeClaim**: Requested 2Gi with matching storage class and access mode.
3. **Pod**: Mounted the PVC at the Apache document root.
4. **Service**: Exposed the pod on NodePort 30008 with proper selector.
5. **Verification**: Confirmed resource status and application accessibility.

### 🎯 Key Success Factors:
- **PersistentVolume**: Bound to PVC
- **PersistentVolumeClaim**: Requests 2Gi and binds correctly
- **Pod**: Runs Apache with volume mounted
- **Service**: Exposes application on NodePort 30008
- **Application Output**: Displays "It works!"

### ⚠️ Critical Configuration Details:
- **PersistentVolume Name**: `pv-datacenter`
- **PersistentVolumeClaim Name**: `pvc-datacenter`
- **Pod Name**: `pod-datacenter`
- **Service Name**: `web-datacenter`
- **NodePort**: 30008
- **Image**: `httpd:latest`
- **Mount Path**: `/usr/local/apache2/htdocs`

### 🔒 Kubernetes Benefits:
- **Persistence**: `hostPath` ensures data storage on the host
- **Reliability**: Pod restart policy ensures uptime
- **Accessibility**: NodePort enables external access
- **Scalability**: PVC allows flexible storage allocation

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Consider using a Deployment instead of a standalone pod for better scalability.

🔐 **Security**: Restrict access to `/mnt/finance` and use read-only mounts if possible.

📊 **Monitoring**: Monitor pod and volume health for errors.

🛡️ **Scalability**: Use a managed storage solution (e.g., NFS, cloud storage) instead of `hostPath` for production.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **PersistentVolume**: Providing storage for applications
- **PersistentVolumeClaim**: Requesting and binding storage
- **Pod**: Running applications with mounted volumes
- **NodePort Service**: Exposing applications externally
- **Apache Web Server**: Serving content from persistent storage

**Kubernetes Features Used:**
- `v1 PersistentVolume` for storage
- `v1 PersistentVolumeClaim` for storage requests
- `v1 Pod` for application execution
- `v1 Service` for NodePort exposure
- `kubectl` for cluster management

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **PersistentVolume** - Created `pv-datacenter` with 4Gi capacity
- **PersistentVolumeClaim** - Created `pvc-datacenter` requesting 2Gi
- **Pod** - Created `pod-datacenter` with Apache and mounted volume
- **Service** - Created `web-datacenter` with NodePort 30008
- **Verification** - Confirmed resources and application accessibility

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The web application is now deployed on the Kubernetes cluster with a persistent volume for storing application code, accessible via NodePort 30008, ready for further enhancements by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Content Updates**: Add files to `/mnt/finance` for serving via Apache
- **Scaling**: Convert to a Deployment for multiple replicas
- **Security**: Apply network policies and storage permissions
- **Monitoring**: Implement health checks and logging