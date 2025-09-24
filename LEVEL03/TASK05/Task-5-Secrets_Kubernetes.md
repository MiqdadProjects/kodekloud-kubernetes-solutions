# 🌟 Task 5 - Deploy Pod with Secret in Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team is deploying tools in a Kubernetes cluster, some of which require secure storage of license information. To achieve this, Kubernetes secrets will be used to securely store sensitive data. The task involves creating a secret from a file and mounting it in a pod for use by a containerized application.

### Requirements:
- Create a generic secret named **`beta`**:
  - Use the content of the file `/opt/beta.txt` on the jump host as the secret value.
  - The secret should contain the password/license-number from `beta.txt`.
- Create a pod named **`secret-devops`** with:
  - A container named **`secret-container-devops`** using the `centos:latest` image.
  - Use a `sleep` command to keep the container running.
  - Mount the `beta` secret as a file under `/opt/apps` in the container.
- Verify the secret by exec-ing into the container and checking the mounted file at `/opt/apps/beta.txt`.
- Ensure the `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Create a Kubernetes secret to store the license key from `beta.txt` and deploy a pod that mounts this secret, ensuring it is accessible within the container.

💡 **Note**: The pod must remain in a running state, and validation may take time, so ensure the pod is running before checking.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- Secret to store license information
- Pod with a single container to consume the secret
- Volume to mount the secret as a file

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Secret**: Stores the license key from `/opt/beta.txt`
- **Pod**: Runs a CentOS container with the secret mounted
- **Volume**: Mounts the secret as a file in the container
- **Container**: Uses a `sleep` command to remain running

### 🎯 Implementation Strategy
1. Create a generic secret from the `beta.txt` file
2. Deploy a pod with a CentOS container and mount the secret
3. Configure the container to stay running with a `sleep` command
4. Verify the pod status and secret accessibility
5. Exec into the container to check the mounted secret file

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Secret and Pod Configuration
Create a file named `secret-devops.yaml`:

```yaml
# secret-devops.yaml
# Define generic secret for license key
apiVersion: v1
kind: Secret
metadata:
  name: beta
type: Opaque
data:
  beta: NWVjdXIz  # base64 encoded value of "5ecur3"
---
# Define pod with secret mounted
apiVersion: v1
kind: Pod
metadata:
  name: secret-devops
  labels:
    app.kubernetes.io/name: secret-devops
spec:
  containers:
    - name: secret-container-devops
      image: centos:latest
      imagePullPolicy: IfNotPresent
      command:
        - /bin/sh
        - -c
      args:
        - sleep 1000
      volumeMounts:
        - mountPath: /opt/apps/beta.txt
          name: secrets
          subPath: beta.txt
  volumes:
    - name: secrets
      secret:
        secretName: beta
        items:
          - key: beta
            path: beta.txt
  restartPolicy: Always
```

**Configuration Breakdown:**
- **Secret**:
  - `name: beta` - Specifies the secret name
  - `type: Opaque` - Generic secret type
  - `data.beta: NWVjdXIz` - Base64-encoded value of the license key (e.g., "5ecur3")
- **Pod**:
  - `name: secret-devops` - Specifies the pod name
  - `labels: app.kubernetes.io/name: secret-devops` - Label for identification
  - `containers.image: centos:latest` - Uses CentOS image
  - `command: ["/bin/sh", "-c"]` and `args: ["sleep 1000"]` - Keeps container running
  - `volumeMounts.mountPath: /opt/apps/beta.txt` - Mounts secret as a file
  - `volumes.secret.secretName: beta` - Links to the `beta` secret
  - `items.key: beta` and `items.path: beta.txt` - Maps secret key to file path

**Note**: The `data.beta: NWVjdXIz` assumes the content of `/opt/beta.txt` is "5ecur3" (base64-encoded as `NWVjdXIz`). If the actual content differs, you must base64-encode the content of `/opt/beta.txt` using:
```bash
cat /opt/beta.txt | base64
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configuration
```bash
kubectl apply -f secret-devops.yaml
```

**Purpose:** Deploy the secret and pod to the Kubernetes cluster.

**Expected Output:**
```
secret/beta created
pod/secret-devops created
```

---

### Step 4: Verify Secret
```bash
kubectl get secret beta
```

**Expected Output:**
```
NAME   TYPE     DATA   AGE
beta   Opaque   1      2m
```

**Verification:** Confirm the secret exists with one data key.

---

### Step 5: Verify Pod
```bash
kubectl get pod secret-devops
```

**Purpose:** Ensure the pod is in a `Running` state.

**Expected Output:**
```
NAME            READY   STATUS    RESTARTS   AGE
secret-devops   1/1     Running   0          2m
```

**Verification:** Confirm the pod is running.

---

### Step 6: Verify Secret Mount
```bash
kubectl exec secret-devops -c secret-container-devops -- cat /opt/apps/beta.txt
```

**Purpose:** Check the mounted secret file in the container.

**Expected Output:**
```
5ecur3
```

**Verification:** Confirm the content of `/opt/apps/beta.txt` matches the original `beta.txt` file.

---

## 🔍 Code Analysis

### Complete Configuration (`secret-devops.yaml`)
```yaml
# Define generic secret for license key
apiVersion: v1
kind: Secret
metadata:
  name: beta
type: Opaque
data:
  beta: NWVjdXIz  # base64 encoded value of "5ecur3"
---
# Define pod with secret mounted
apiVersion: v1
kind: Pod
metadata:
  name: secret-devops
  labels:
    app.kubernetes.io/name: secret-devops
spec:
  containers:
    - name: secret-container-devops
      image: centos:latest
      imagePullPolicy: IfNotPresent
      command:
        - /bin/sh
        - -c
      args:
        - sleep 1000
      volumeMounts:
        - mountPath: /opt/apps/beta.txt
          name: secrets
          subPath: beta.txt
  volumes:
    - name: secrets
      secret:
        secretName: beta
        items:
          - key: beta
            path: beta.txt
  restartPolicy: Always
```

### Secret Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | beta | Name of the secret |
| **type** | Opaque | Generic secret type |
| **data.beta** | NWVjdXIz | Base64-encoded license key (e.g., "5ecur3") |

### Pod Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | secret-devops | Name of the pod |
| **labels** | app.kubernetes.io/name: secret-devops | Label for identification |
| **containers.image** | centos:latest | CentOS image |
| **command** | ["/bin/sh", "-c"] | Shell command |
| **args** | ["sleep 1000"] | Keeps container running |
| **volumeMounts.mountPath** | /opt/apps/beta.txt | Mounts secret as a file |
| **volumes.secret.secretName** | beta | Links to secret |

---

## ✅ Verification Steps

### Step 1: Verify Secret
```bash
kubectl get secret beta
```

**Expected Output:**
```
NAME   TYPE     DATA   AGE
beta   Opaque   1      2m
```

### Step 2: Verify Pod
```bash
kubectl get pod secret-devops
```

**Expected Output:**
```
NAME            READY   STATUS    RESTARTS   AGE
secret-devops   1/1     Running   0          2m
```

### Step 3: Verify Secret Mount
```bash
kubectl exec secret-devops -c secret-container-devops -- cat /opt/apps/beta.txt
```

**Expected Output:**
```
5ecur3
```

---

## 🧪 Testing

### Verify Secret Content
```bash
kubectl describe secret beta
```

**Purpose:** Confirm the secret contains the expected key.

### Verify Pod Status
```bash
kubectl describe pod secret-devops
```

**Purpose:** Check pod events for issues with secret mounting or container startup.

### Verify File Accessibility
```bash
kubectl exec secret-devops -c secret-container-devops -- ls /opt/apps
```

**Expected Output:**
```
beta.txt
```

**Purpose:** Confirm the secret file is mounted correctly.

---

## 📚 Quick Command Reference

```bash
# Apply configuration
kubectl apply -f secret-devops.yaml

# Verify resources
kubectl get secret beta
kubectl get pod secret-devops

# Verify secret mount
kubectl exec secret-devops -c secret-container-devops -- cat /opt/apps/beta.txt
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Secret Not Created
**Symptoms:** Secret is missing or contains incorrect data.
**Solution:** Verify the base64-encoded value in the YAML.
```bash
kubectl describe secret beta
cat /opt/beta.txt | base64
```

### Issue 2: Pod Not Running
**Symptoms:** Pod in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod secret-devops
kubectl logs secret-devops -c secret-container-devops
```

### Issue 3: Secret Not Mounted
**Symptoms:** `/opt/apps/beta.txt` is missing or empty.
**Solution:** Verify volume mount and secret configuration.
```bash
kubectl exec secret-devops -c secret-container-devops -- ls /opt/apps
kubectl describe pod secret-devops
```

### Issue 4: Incorrect Secret Data
**Symptoms:** Mounted file contains wrong data.
**Solution:** Recreate the secret with correct base64-encoded value.
```bash
kubectl delete secret beta
cat /opt/beta.txt | base64
# Update secret-devops.yaml with correct base64 value and reapply
kubectl apply -f secret-devops.yaml
```

---

## 💡 Additional Tips

- **Secret Encoding**: Always base64-encode the content of `/opt/beta.txt` to ensure accuracy.
  ```bash
  cat /opt/beta.txt | base64
  ```
- **Pod Logs**: Check logs if the container fails to start.
  ```bash
  kubectl logs secret-devops -c secret-container-devops
  ```
- **Dry Run**: Test configuration before applying.
  ```bash
  kubectl apply -f secret-devops.yaml --dry-run=client
  ```
- **File Verification**: Confirm the mounted file exists.
  ```bash
  kubectl exec secret-devops -c secret-container-devops -- cat /opt/apps/beta.txt
  ```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Secret Creation**: Correctly encoding and storing the license key from `beta.txt`.
2. **Secret Mounting**: Mounting the secret as a file at the correct path.
3. **Container Persistence**: Ensuring the container remains running with a `sleep` command.
4. **Verification**: Accessing the mounted secret file in the container.

### 💡 Solution Approach:
1. **Secret**: Created `beta` with base64-encoded content from `beta.txt`.
2. **Pod**: Configured `secret-devops` with a CentOS container and mounted secret.
3. **Command**: Used `sleep 1000` to keep the container running.
4. **Verification**: Checked pod status and secret file content.

### 🎯 Key Success Factors:
- **Secret**: Contains correct license key
- **Pod**: Runs with secret mounted at `/opt/apps/beta.txt`
- **Container**: Stays running with `sleep` command
- **Verification**: Secret file content is accessible

### ⚠️ Critical Configuration Details:
- **Secret Name**: `beta`
- **Pod Name**: `secret-devops`
- **Container Name**: `secret-container-devops`
- **Image**: `centos:latest`
- **Mount Path**: `/opt/apps/beta.txt`
- **Command**: `sleep 1000`

### 🔒 Kubernetes Benefits:
- **Security**: Secrets store sensitive data securely
- **Reliability**: Pod restart policy ensures uptime
- **Flexibility**: Secret mounted as a file for application use
- **Debugging**: Easy to verify with `kubectl exec`

---

## ⚠️ Important Production Notes

🔧 **Secret Management**: Use sealed secrets or external secret management for production.

🔐 **Security**: Restrict access to secrets with RBAC policies.

📊 **Monitoring**: Monitor pod health and secret usage.

🛡️ **Scalability**: Consider using a Deployment for multiple replicas.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **Secrets**: Storing sensitive data securely
- **Pod**: Running containers with mounted secrets
- **Volume Mounts**: Mounting secrets as files
- **Container Management**: Keeping containers running with `sleep`

**Kubernetes Features Used:**
- `v1 Secret` for secure storage
- `v1 Pod` for application execution
- `volumeMounts` for secret access
- `kubectl exec` for verification

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **Secret** - Created `beta` with license key from `beta.txt`
- **Pod** - Created `secret-devops` with CentOS container
- **Secret Mount** - Mounted secret at `/opt/apps/beta.txt`
- **Verification** - Confirmed pod status and secret content

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The pod with a mounted secret is now running on the Kubernetes cluster, with the license key accessible at `/opt/apps/beta.txt`, ready for further use by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Tool Integration**: Use the license key in applications
- **Security**: Enhance with secret encryption
- **Monitoring**: Add health checks and logging
- **Scaling**: Convert to a Deployment for redundancy