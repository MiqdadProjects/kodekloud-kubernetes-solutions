**🌟 Task 10 - Create Time Check Pod with ConfigMap and Logging**

**📌 Task Description**

The Nautilus DevOps team is developing a pod to periodically log timestamps for testing purposes, with potential use in existing clusters. The pod must use a ConfigMap for configuration and log output to a specific volume.

**Requirements:**
- Create a pod named `time-check` in the `datacenter` namespace.
- Use container name `time-check` with `busybox:latest` image (specify the tag).
- Create a ConfigMap named `time-config` in the `datacenter` namespace with data `TIME_FREQ=11`.
- Run the command: `while true; do date; sleep $TIME_FREQ; done` and write output to `/opt/security/time/time-check.log`.
- Set an environment variable `TIME_FREQ` sourced from the ConfigMap.
- Create a volume `log-volume` and mount it at `/opt/security/time`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like namespaces or file paths may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy a pod with ConfigMap-driven logging and persistent volume.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host where `kubectl` is configured to interact with the Kubernetes cluster.

---

## 🔹 Step 2: Create Manifest for Namespace, ConfigMap, and Pod

```bash
vi time-check.yaml
```

**Purpose**: Define the namespace, ConfigMap, and pod with volume and logging configuration.

**YAML Content**:
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: datacenter
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: datacenter
data:
  TIME_FREQ: "11"
---
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: datacenter
spec:
  containers:
    - name: time-check
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      envFrom:
        - configMapRef:
            name: time-config
      volumeMounts:
        - mountPath: /opt/security/time
          name: log-volume
      command:
        - /bin/sh
        - -c
      args:
        - while true; do date; sleep $TIME_FREQ; done > /opt/security/time/time-check.log
  restartPolicy: Always
  volumes:
    - name: log-volume
      emptyDir: {}
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Namespace`: Creates the `datacenter` namespace.
- `kind: ConfigMap`: Defines `time-config` with `TIME_FREQ=11`.
- `kind: Pod`: Creates the `time-check` pod in the `datacenter` namespace.
- `envFrom`: Injects `TIME_FREQ` from the ConfigMap as an environment variable.
- `volumeMounts`: Mounts `log-volume` at `/opt/security/time`.
- `command` and `args`: Runs a loop to log timestamps every 11 seconds.
- `volumes`: Defines an `emptyDir` volume for logging.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.
- `restartPolicy: Always`: Ensures pod restarts on failure.

---

## 🔹 Step 3: Apply the Manifest

```bash
kubectl apply -f time-check.yaml
```

**Purpose**: Deploy the namespace, ConfigMap, and pod.

**Expected Output**:
```
namespace/datacenter created
configmap/time-config created
pod/time-check created
```

---

## 🔹 Step 4: Verify Namespace

```bash
kubectl get namespaces
```

**Purpose**: Confirm the `datacenter` namespace exists.

**Expected Output**:
```
NAME              STATUS   AGE
datacenter        Active   10s
...
```

---

## 🔹 Step 5: Verify ConfigMap

```bash
kubectl get configmap -n datacenter
```

**Purpose**: Confirm the `time-config` ConfigMap is created.

**Expected Output**:
```
NAME         DATA   AGE
time-config  1      10s
```

**Details**:
```bash
kubectl describe configmap time-config -n datacenter
```

**Expected Output (partial)**:
```
Name:         time-config
Namespace:    datacenter
Data
====
TIME_FREQ:    11
```

---

## 🔹 Step 6: Verify Pod Status

```bash
kubectl get pods -n datacenter
```

**Purpose**: Ensure the `time-check` pod is running.

**Expected Output**:
```
NAME         READY   STATUS    RESTARTS   AGE
time-check   1/1     Running   0          10s
```

---

## 🔹 Step 7: Verify Log Output

```bash
kubectl exec -n datacenter time-check -- cat /opt/security/time/time-check.log
```

**Purpose**: Check the timestamp logs in the volume.

**Expected Output**:
```
Thu Sep 18 16:59:00 UTC 2025
Thu Sep 18 16:59:11 UTC 2025
...
```

---

## 🔹 Step 8: Verify Environment Variable

```bash
kubectl exec -n datacenter time-check -- env | grep TIME_FREQ
```

**Purpose**: Confirm the `TIME_FREQ` environment variable is set.

**Expected Output**:
```
TIME_FREQ=11
```

---

## 📋 Quick Command Reference

```bash
# Create and apply manifest
vi time-check.yaml
# (Paste YAML content)
kubectl apply -f time-check.yaml

# Verify resources
kubectl get namespaces
kubectl get configmap -n datacenter
kubectl get pods -n datacenter
kubectl exec -n datacenter time-check -- cat /opt/security/time/time-check.log
kubectl exec -n datacenter time-check -- env | grep TIME_FREQ
```

---

## 💡 Additional Tips

- **Task Variability**: Namespace, pod names, or file paths may differ; adapt the YAML accordingly.
- **ConfigMap Usage**: Use ConfigMaps for reusable configuration data.
- **Volume Management**: `emptyDir` is temporary; use PersistentVolumes for production.
- **Dry Run**: Test with `kubectl apply -f time-check.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Namespace Not Found**
**Symptoms**: `Error: namespace "datacenter" not found`.
**Solution**: Ensure the namespace is created.
```bash
kubectl create namespace datacenter
```

### **Issue 2: Pod Not Running**
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod time-check -n datacenter
kubectl logs time-check -n datacenter
```

### **Issue 3: Log File Not Created**
**Symptoms**: No output in `/opt/security/time/time-check.log`.
**Solution**: Verify command and volume mount.
```bash
kubectl describe pod time-check -n datacenter
# Check volumeMounts and command
```

### **Issue 4: Environment Variable Missing**
**Symptoms**: `TIME_FREQ` not set.
**Solution**: Verify ConfigMap reference.
```bash
kubectl describe pod time-check -n datacenter
# Ensure envFrom references time-config
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **integrating a ConfigMap with a pod** to configure a looping timestamp command and log output to a volume-mounted path.

**💡 Solution Approach:**
1. **Namespace and ConfigMap**: Created `datacenter` namespace and `time-config` ConfigMap.
2. **Pod Configuration**: Defined pod with `busybox:latest`, `envFrom`, and volume mount.
3. **Logging Command**: Configured `while` loop to log timestamps every 11 seconds.
4. **Verification**: Checked pod status, logs, and environment variables.
5. **Best Practices**: Used `emptyDir` volume and `imagePullPolicy: IfNotPresent`.

**🎯 Key Success Factors:**
- **Correct namespace**: Used `datacenter` for isolation.
- **ConfigMap integration**: `TIME_FREQ` sourced correctly.
- **Logging**: Timestamps written to specified path.
- **Verification**: Confirmed pod, ConfigMap, and log output.

**⚠️ Critical Configuration Details:**
- **Namespace**: Must be `datacenter`.
- **Pod Name**: Must be `time-check`.
- **Container Name**: Must be `time-check`.
- **Image**: Must be `busybox:latest`.
- **ConfigMap**: Must have `TIME_FREQ=11`.
- **Volume Mount**: Must be `/opt/security/time`.

**🔒 Kubernetes Benefits:**
- **Configuration Management**: ConfigMaps decouple configuration from pods.
- **Logging**: Volume mounts enable persistent logging.
- **Reliability**: `restartPolicy: Always` ensures pod recovery.

---

## ⚠️ Important Production Notes

🔧 **Logging**: Use PersistentVolumes for durable storage in production.

🔐 **Security**: Restrict ConfigMap access with RBAC.

📊 **Monitoring**: Monitor log file growth and pod resource usage.

🛡️ **Scalability**: Consider Job or CronJob for recurring tasks.

---
