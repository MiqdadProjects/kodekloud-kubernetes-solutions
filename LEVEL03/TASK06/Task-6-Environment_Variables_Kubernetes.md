# 🌟 Task 6 - Deploy Pod with Environment Variables in Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team needs to configure applications in a Kubernetes cluster by defining environment variables to expose specific pod and cluster metadata. These variables will be used within application configurations. The task involves creating a pod with a container that uses environment variables sourced from Kubernetes field references.

### Requirements:
- Create a pod named **`envars`** with:
  - A container named **`fieldref-container`** using the `httpd:latest` image.
  - Command: `sh`, `-c`, and args: `while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;`.
  - Four environment variables:
    - **`NODE_NAME`**: Sourced from `spec.nodeName` using `valueFrom.fieldRef`.
    - **`POD_NAME`**: Sourced from `metadata.name` using `valueFrom.fieldRef`.
    - **`POD_IP`**: Sourced from `status.podIP` using `valueFrom.fieldRef`.
    - **`POD_SERVICE_ACCOUNT`**: Sourced from `spec.serviceAccountName` using `valueFrom.fieldRef`.
  - Restart policy set to `Never`.
- Verify the environment variables by exec-ing into the pod and using the `printenv` command.
- Ensure the `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Deploy a pod in the Kubernetes cluster with environment variables sourced from field references, ensuring they are accessible within the container.

💡 **Note**: The pod’s container continuously prints the environment variables every 10 seconds, and the `Never` restart policy ensures the pod does not restart on failure.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- Pod with a single container
- Environment variables sourced from Kubernetes metadata
- Container command to print environment variables

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Pod**: Runs a container with environment variables
- **Container**: Uses Apache image and prints environment variables
- **Environment Variables**: Sourced from Kubernetes field references
- **Command**: Continuously outputs variable values

### 🎯 Implementation Strategy
1. Create a pod with the specified name and container configuration
2. Define environment variables using field references
3. Set the restart policy to `Never`
4. Verify the pod status and environment variables
5. Exec into the container to check variable output

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Pod Configuration
Create a file named `envars-pod.yaml`:

```yaml
# envars-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars
spec:
  containers:
    - name: fieldref-container
      image: httpd:latest
      imagePullPolicy: IfNotPresent
      command:
        - /bin/sh
        - -c
      args:
        - while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never
```

**Configuration Breakdown:**
- **Pod**:
  - `name: envars` - Specifies the pod name
  - `restartPolicy: Never` - Prevents automatic restarts on failure
- **Container**:
  - `name: fieldref-container` - Container name
  - `image: httpd:latest` - Uses Apache image
  - `imagePullPolicy: IfNotPresent` - Pulls image only if not present
  - `command: ["/bin/sh", "-c"]` and `args: ["while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;"]` - Prints environment variables every 10 seconds
- **Environment Variables**:
  - `NODE_NAME`: Sourced from `spec.nodeName` (node running the pod)
  - `POD_NAME`: Sourced from `metadata.name` (pod name, `envars`)
  - `POD_IP`: Sourced from `status.podIP` (pod’s IP address)
  - `POD_SERVICE_ACCOUNT`: Sourced from `spec.serviceAccountName` (service account assigned to the pod)

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configuration
```bash
kubectl apply -f envars-pod.yaml
```

**Purpose:** Deploy the pod to the Kubernetes cluster.

**Expected Output:**
```
pod/envars created
```

---

### Step 4: Verify Pod
```bash
kubectl get pod envars
```

**Purpose:** Ensure the pod is in a `Running` state.

**Expected Output:**
```
NAME     READY   STATUS    RESTARTS   AGE
envars   1/1     Running   0          2m
```

**Verification:** Confirm the pod is running.

---

### Step 5: Verify Environment Variables
```bash
kubectl exec envars -c fieldref-container -- printenv
```

**Purpose:** Check the environment variables in the container.

**Expected Output (example):**
```
NODE_NAME=node01
POD_NAME=envars
POD_IP=10.244.0.5
POD_SERVICE_ACCOUNT=default
...
```

**Verification:** Confirm the variables `NODE_NAME`, `POD_NAME`, `POD_IP`, and `POD_SERVICE_ACCOUNT` are set with appropriate values.

---

### Step 6: Check Container Logs
```bash
kubectl logs envars -c fieldref-container
```

**Purpose:** Verify the container is printing the environment variables.

**Expected Output (example):**
```
/n
node01 envars
10.244.0.5 default
/n
node01 envars
10.244.0.5 default
...
```

**Verification:** Confirm the logs show the environment variables printed every 10 seconds.

---

## 🔍 Code Analysis

### Complete Configuration (`envars-pod.yaml`)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars
spec:
  containers:
    - name: fieldref-container
      image: httpd:latest
      imagePullPolicy: IfNotPresent
      command:
        - /bin/sh
        - -c
      args:
        - while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never
```

### Pod Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | envars | Name of the pod |
| **restartPolicy** | Never | Prevents automatic restarts |
| **containers.name** | fieldref-container | Container name |
| **containers.image** | httpd:latest | Apache image |
| **command** | ["/bin/sh", "-c"] | Shell command |
| **args** | ["while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;"] | Prints variables every 10 seconds |

### Environment Variables
| Name | Source (`fieldPath`) | Description |
|------|----------------------|-------------|
| **NODE_NAME** | spec.nodeName | Name of the node running the pod |
| **POD_NAME** | metadata.name | Name of the pod (`envars`) |
| **POD_IP** | status.podIP | IP address of the pod |
| **POD_SERVICE_ACCOUNT** | spec.serviceAccountName | Service account assigned to the pod |

---

## ✅ Verification Steps

### Step 1: Verify Pod
```bash
kubectl get pod envars
```

**Expected Output:**
```
NAME     READY   STATUS    RESTARTS   AGE
envars   1/1     Running   0          2m
```

### Step 2: Verify Environment Variables
```bash
kubectl exec envars -c fieldref-container -- printenv
```

**Expected Output (example):**
```
NODE_NAME=node01
POD_NAME=envars
POD_IP=10.244.0.5
POD_SERVICE_ACCOUNT=default
...
```

### Step 3: Verify Logs
```bash
kubectl logs envars -c fieldref-container
```

**Expected Output (example):**
```
/n
node01 envars
10.244.0.5 default
/n
node01 envars
10.244.0.5 default
...
```

---

## 🧪 Testing

### Verify Pod Status
```bash
kubectl describe pod envars
```

**Purpose:** Check pod events for issues with container startup or environment variables.

### Verify Environment Variables
```bash
kubectl exec envars -c fieldref-container -- printenv NODE_NAME POD_NAME POD_IP POD_SERVICE_ACCOUNT
```

**Purpose:** Confirm the specific environment variables are set correctly.

### Verify Command Output
```bash
kubectl logs envars -c fieldref-container
```

**Purpose:** Ensure the container is printing the variables as expected.

---

## 📚 Quick Command Reference

```bash
# Apply configuration
kubectl apply -f envars-pod.yaml

# Verify pod and environment variables
kubectl get pod envars
kubectl exec envars -c fieldref-container -- printenv
kubectl logs envars -c fieldref-container
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pod Not Running
**Symptoms:** Pod in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod envars
kubectl logs envars -c fieldref-container
```

### Issue 2: Environment Variables Not Set
**Symptoms:** `printenv` shows missing or incorrect variables.
**Solution:** Verify environment variable configuration in YAML.
```bash
kubectl get pod envars -o yaml
```

### Issue 3: Command Not Executing
**Symptoms:** Logs show no output or errors.
**Solution:** Verify command and args syntax.
```bash
kubectl describe pod envars
kubectl logs envars -c fieldref-container
```

### Issue 4: Incorrect Field References
**Symptoms:** Variables are empty or undefined.
**Solution:** Ensure `fieldPath` values are correct (`spec.nodeName`, `metadata.name`, etc.).
```bash
kubectl get pod envars -o yaml
```

---

## 💡 Additional Tips

- **Pod Logs**: Check logs for command execution issues.
  ```bash
  kubectl logs envars -c fieldref-container
  ```
- **Dry Run**: Test configuration before applying.
  ```bash
  kubectl apply -f envars-pod.yaml --dry-run=client
  ```
- **Variable Verification**: Use specific `printenv` commands for debugging.
  ```bash
  kubectl exec envars -c fieldref-container -- printenv NODE_NAME
  ```
- **Restart Policy**: The `Never` policy means manual intervention is needed if the pod fails.

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Environment Variables**: Correctly sourcing variables from Kubernetes field references.
2. **Command Syntax**: Ensuring the command and args format is correct for continuous output.
3. **Restart Policy**: Setting `Never` as specified.
4. **Verification**: Accessing and confirming environment variable values.

### 💡 Solution Approach:
1. **Environment Variables**: Configured using `valueFrom.fieldRef` with correct `fieldPath` values.
2. **Command**: Used `sh -c` with a `while` loop to print variables.
3. **Pod**: Set `restartPolicy: Never` and used `httpd:latest` image.
4. **Verification**: Used `kubectl exec` and `logs` to confirm output.

### 🎯 Key Success Factors:
- **Environment Variables**: Correctly sourced from Kubernetes metadata
- **Command**: Continuously prints variables every 10 seconds
- **Pod**: Runs without restarting
- **Verification**: Variables are accessible via `printenv`

### ⚠️ Critical Configuration Details:
- **Pod Name**: `envars`
- **Container Name**: `fieldref-container`
- **Image**: `httpd:latest`
- **Restart Policy**: `Never`
- **Environment Variables**: `NODE_NAME`, `POD_NAME`, `POD_IP`, `POD_SERVICE_ACCOUNT`

### 🔒 Kubernetes Benefits:
- **Dynamic Configuration**: Field references provide real-time metadata
- **Reliability**: Pod runs continuously for verification
- **Debugging**: Easy access to variables via `kubectl exec`
- **Flexibility**: Environment variables can be used in application configs

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Consider using a Deployment for automatic pod management.

🔐 **Security**: Restrict access to sensitive metadata if exposed.

📊 **Monitoring**: Monitor pod logs for unexpected behavior.

🛡️ **Scalability**: Adjust for multiple pods if needed for production.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **Environment Variables**: Sourcing from Kubernetes field references
- **Pod**: Running containers with custom commands
- **Field References**: Accessing pod and cluster metadata
- **Verification**: Using `kubectl exec` and `logs` for debugging

**Kubernetes Features Used:**
- `v1 Pod` for application execution
- `env.valueFrom.fieldRef` for dynamic variables
- `kubectl exec` and `logs` for verification

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **Pod** - Created `envars` with `fieldref-container`
- **Environment Variables** - Configured `NODE_NAME`, `POD_NAME`, `POD_IP`, `POD_SERVICE_ACCOUNT`
- **Command** - Set to print variables every 10 seconds
- **Restart Policy** - Set to `Never`
- **Verification** - Confirmed variables via `printenv` and logs

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The pod with environment variables is now running on the Kubernetes cluster, continuously printing metadata, ready for further use by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Configuration**: Use variables in application logic
- **Monitoring**: Add logging and health checks
- **Security**: Restrict metadata exposure
- **Scaling**: Convert to a Deployment for redundancy