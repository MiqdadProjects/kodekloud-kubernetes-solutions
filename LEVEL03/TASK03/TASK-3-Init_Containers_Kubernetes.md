# 🌟 Task 3 - Deploy Application with Init Container on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team needs to deploy applications on a Kubernetes cluster with specific pre-requisite configurations that cannot be made within the container images. To address this, they plan to use init containers to perform initial setup tasks before the main application container starts. This task involves testing a sample scenario with a deployment that includes an init container to create a configuration file, which the main container will use.

### Requirements:
- Create a deployment named **`ic-deploy-xfusion`** with:
  - 1 replica.
  - Labels `app: ic-xfusion` for both the deployment selector and pod template metadata.
- Configure an init container named **`ic-msg-xfusion`**:
  - Use the `ubuntu:latest` image.
  - Run the command `/bin/bash`, `-c`, `echo Init Done - Welcome to xFusionCorp Industries > /ic/official`.
  - Mount a volume named `ic-volume-xfusion` at `/ic`.
- Configure a main container named **`ic-main-xfusion`**:
  - Use the `ubuntu:latest` image.
  - Run the command `/bin/bash`, `-c`, `while true; do cat /ic/official; sleep 5; done`.
  - Mount a volume named `ic-volume-xfusion` at `/ic`.
- Create a volume named **`ic-volume-xfusion`** of type `emptyDir`.
- Ensure the `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Deploy an application on the Kubernetes cluster using an init container to set up a configuration file, which the main container will read and display repeatedly.

💡 **Note**: The init container creates a file that the main container uses, leveraging a shared `emptyDir` volume for communication.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- Deployment with one replica
- Init container for initial configuration
- Main container to display configuration
- `emptyDir` volume for sharing data between containers

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Deployment**: Manages a single pod with an init container and a main container
- **Init Container**: Creates a configuration file in a shared volume
- **Main Container**: Reads and displays the configuration file continuously
- **Volume**: Uses `emptyDir` to share data between containers
- **Labels**: Ensures proper pod selection and management

### 🎯 Implementation Strategy
1. Create a deployment with the specified name and labels
2. Configure an init container to write a message to a file
3. Configure a main container to read and display the file
4. Use an `emptyDir` volume to share the file between containers
5. Verify the deployment, pod status, and container logs

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Deployment Configuration
Create a file named `ic-deploy.yaml`:

```yaml
# ic-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      initContainers:
      - name: ic-msg-xfusion
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/official"]
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      containers:
      - name: ic-main-xfusion
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "while true; do cat /ic/official; sleep 5; done"]
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      volumes:
      - name: ic-volume-xfusion
        emptyDir: {}
```

**Configuration Breakdown:**
- **Deployment**:
  - `name: ic-deploy-xfusion` - Specifies the deployment name
  - `replicas: 1` - Runs one pod
  - `selector` and `labels` - Matches pods using `app: ic-xfusion`
- **Init Container**:
  - `name: ic-msg-xfusion` - Init container name
  - `image: ubuntu:latest` - Uses the specified image
  - `command`: Writes a message to `/ic/official`
  - `volumeMounts`: Mounts `ic-volume-xfusion` at `/ic`
- **Main Container**:
  - `name: ic-main-xfusion` - Main container name
  - `image: ubuntu:latest` - Uses the specified image
  - `command`: Continuously reads and displays `/ic/official`
  - `volumeMounts`: Mounts `ic-volume-xfusion` at `/ic`
- **Volume**:
  - `name: ic-volume-xfusion` - Shared volume
  - `emptyDir: {}` - Temporary storage that exists for the pod’s lifetime

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configuration
```bash
kubectl apply -f ic-deploy.yaml
```

**Purpose:** Deploy the configuration to the Kubernetes cluster.

**Expected Output:**
```
deployment.apps/ic-deploy-xfusion created
```

---

### Step 4: Verify Deployment and Pods
```bash
kubectl get deploy ic-deploy-xfusion
```

**Purpose:** Check the status of the `ic-deploy-xfusion` deployment.

**Expected Output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-xfusion  1/1     1            1           1m
```

**Verification:** Confirm the deployment shows `1/1` ready replicas.

```bash
kubectl get pods -l app=ic-xfusion
```

**Purpose:** Ensure the pod is in a `Running` state.

**Expected Output:**
```
NAME                               READY   STATUS    RESTARTS   AGE
ic-deploy-xfusion-65cdbff969-sb744  1/1     Running   0          1m
```

**Verification:** Confirm the pod is in `Running` state.

---

### Step 5: Check Container Logs
```bash
kubectl logs -l app=ic-xfusion -c ic-main-xfusion
```

**Purpose:** Verify the main container is reading and displaying the file created by the init container.

**Expected Output:**
```
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
```

**Verification:** Confirm the logs show the repeated message, indicating the init container successfully created the file and the main container is reading it.

---

## 🔍 Code Analysis

### Complete Configuration (`ic-deploy.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      initContainers:
      - name: ic-msg-xfusion
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/official"]
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      containers:
      - name: ic-main-xfusion
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "while true; do cat /ic/official; sleep 5; done"]
        volumeMounts:
        - name: ic-volume-xfusion
          mountPath: /ic
      volumes:
      - name: ic-volume-xfusion
        emptyDir: {}
```

### Deployment Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | ic-deploy-xfusion | Name of the deployment |
| **replicas** | 1 | Number of pod replicas |
| **selector** | app: ic-xfusion | Matches pods to the deployment |
| **template labels** | app: ic-xfusion | Labels for the pod |

### Init Container Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | ic-msg-xfusion | Name of the init container |
| **image** | ubuntu:latest | Ubuntu image with latest tag |
| **command** | ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/official"] | Writes message to file |
| **volumeMount** | ic-volume-xfusion at /ic | Mounts shared volume |

### Main Container Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | ic-main-xfusion | Name of the main container |
| **image** | ubuntu:latest | Ubuntu image with latest tag |
| **command** | ["/bin/bash", "-c", "while true; do cat /ic/official; sleep 5; done"] | Reads and displays file repeatedly |
| **volumeMount** | ic-volume-xfusion at /ic | Mounts shared volume |

### Volume Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | ic-volume-xfusion | Name of the volume |
| **type** | emptyDir | Temporary storage for pod lifetime |

---

## ✅ Verification Steps

### Step 1: Verify Deployment
```bash
kubectl get deploy ic-deploy-xfusion
```

**Expected Output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-xfusion  1/1     1            1           2m
```

### Step 2: Verify Pods
```bash
kubectl get pods -l app=ic-xfusion
```

**Expected Output:**
```
NAME                               READY   STATUS    RESTARTS   AGE
ic-deploy-xfusion-65cdbff969-sb744  1/1     Running   0          2m
```

### Step 3: Verify Logs
```bash
kubectl logs -l app=ic-xfusion -c ic-main-xfusion
```

**Expected Output:**
```
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
```

---

## 🧪 Testing

### Verify Init Container Execution
```bash
kubectl describe pod -l app=ic-xfusion
```

**Purpose:** Check events to confirm the init container completed successfully.

### Verify File Creation
```bash
POD=$(kubectl get pod -l app=ic-xfusion -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -c ic-main-xfusion -- cat /ic/official
```

**Expected Output:**
```
Init Done - Welcome to xFusionCorp Industries
```

**Purpose:** Confirm the file created by the init container exists in the shared volume.

### Verify Container Command
```bash
kubectl logs -l app=ic-xfusion -c ic-main-xfusion
```

**Purpose:** Ensure the main container is reading and displaying the file content every 5 seconds.

---

## 📚 Quick Command Reference

```bash
# Apply configuration
kubectl apply -f ic-deploy.yaml

# Verify deployment, pods, and logs
kubectl get deploy ic-deploy-xfusion
kubectl get pods -l app=ic-xfusion
kubectl logs -l app=ic-xfusion -c ic-main-xfusion
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms:** Pods in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod -l app=ic-xfusion
kubectl logs -l app=ic-xfusion -c ic-msg-xfusion
kubectl logs -l app=ic-xfusion -c ic-main-xfusion
```

### Issue 2: Init Container Failure
**Symptoms:** Pod stuck in `Init` status.
**Solution:** Verify init container command and volume mount.
```bash
kubectl describe pod -l app=ic-xfusion
kubectl logs -l app=ic-xfusion -c ic-msg-xfusion
```

### Issue 3: File Not Found in Main Container
**Symptoms:** Main container logs show errors or no output.
**Solution:** Ensure the `emptyDir` volume is correctly mounted.
```bash
kubectl exec $POD -c ic-main-xfusion -- ls /ic
```

### Issue 4: Incorrect Labels
**Symptoms:** Deployment doesn’t select pods.
**Solution:** Verify selector and template labels match.
```bash
kubectl get deploy ic-deploy-xfusion -o yaml
```

---

## 💡 Additional Tips

- **Pod Logs**: Check init container logs for setup issues.
  ```bash
  kubectl logs -l app=ic-xfusion -c ic-msg-xfusion
  ```
- **Dry Run**: Test configuration before applying.
  ```bash
  kubectl apply -f ic-deploy.yaml --dry-run=client
  ```
- **Volume Verification**: Confirm the file exists in the shared volume.
  ```bash
  kubectl exec $POD -c ic-main-xfusion -- cat /ic/official
  ```
- **Debugging**: Use `kubectl describe` to inspect pod events for errors.

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Init Container Setup**: Correctly configuring the init container to write to a shared volume.
2. **Volume Sharing**: Using `emptyDir` to share data between init and main containers.
3. **Command Syntax**: Ensuring correct command syntax for both containers.
4. **Label Consistency**: Matching labels across deployment and pod template.

### 💡 Solution Approach:
1. **Init Container**: Configured to write a message to `/ic/official` using a shared volume.
2. **Main Container**: Set to read and display the file continuously.
3. **Volume**: Used `emptyDir` to enable file sharing between containers.
4. **Labels**: Ensured consistent `app: ic-xfusion` labels for pod selection.
5. **Verification**: Checked pod status and logs to confirm functionality.

### 🎯 Key Success Factors:
- **Init Container**: Successfully creates `/ic/official`
- **Main Container**: Continuously displays the message
- **Volume**: `emptyDir` enables file sharing
- **Logs**: Show repeated output of "Init Done - Welcome to xFusionCorp Industries"

### ⚠️ Critical Configuration Details:
- **Deployment Name**: `ic-deploy-xfusion`
- **Replicas**: 1
- **Labels**: `app: ic-xfusion`
- **Init Container**: `ic-msg-xfusion`, `ubuntu:latest`
- **Main Container**: `ic-main-xfusion`, `ubuntu:latest`
- **Volume**: `ic-volume-xfusion`, `emptyDir`

### 🔒 Kubernetes Benefits:
- **Initialization**: Init containers ensure setup before the main application
- **Isolation**: Containers share data via `emptyDir` volume
- **Reliability**: Kubernetes manages pod lifecycle
- **Debugging**: Logs and events provide clear diagnostics

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for updates with minimal disruption.

🔐 **Security**: Restrict container privileges and use read-only mounts where possible.

📊 **Monitoring**: Monitor pod health and log output for errors.

🛡️ **Scalability**: Adjust replicas for higher availability if needed.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **Init Containers**: Performing setup tasks before main container execution
- **EmptyDir Volume**: Sharing data between containers in a pod
- **Deployment**: Managing application pods
- **Labels**: Selecting and organizing Kubernetes resources

**Kubernetes Features Used:**
- `apps/v1 Deployment` for pod management
- `initContainers` for setup tasks
- `emptyDir` volume for temporary storage
- `kubectl logs` for debugging

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **Deployment** - Created `ic-deploy-xfusion` with 1 replica
- **Init Container** - Configured `ic-msg-xfusion` to write message
- **Main Container** - Configured `ic-main-xfusion` to display message
- **Volume** - Used `emptyDir` named `ic-volume-xfusion`
- **Verification** - Confirmed pod status and log output

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The deployment with an init container is now running on the Kubernetes cluster, with the init container creating a configuration file and the main container displaying its contents, ready for further use by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Scaling**: Increase replicas for redundancy
- **Monitoring**: Add health checks and logging
- **Security**: Apply pod security policies
- **Enhancements**: Extend init container tasks for more complex setups