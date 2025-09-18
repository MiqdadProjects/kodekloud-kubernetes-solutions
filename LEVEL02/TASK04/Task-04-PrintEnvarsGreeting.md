**🌟 Task 4 - Create Pod with Environment Variables**

**📌 Task Description**

The Nautilus DevOps team is testing an application that sends greetings using environment variables. A pod needs to be created to echo these variables.

**Requirements:**
- Create a pod named `print-envars-greeting`.
- Container: Named `print-env-container`, use `bash:latest` image.
- Environment variables:
  - `GREETING`: "Welcome to"
  - `COMPANY`: "Nautilus"
  - `GROUP`: "Ltd"
- Command: `["/bin/sh", "-c", "echo $(GREETING) $(COMPANY) $(GROUP)"]`.
- Set `restartPolicy: Never`.
- Verify output with `kubectl logs -f print-envars-greeting`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names or variables may differ in your environment.

👉 **Your task**: Deploy a pod that echoes environment variables and completes.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Pod Manifest

```bash
vi pod.yaml
```

**Purpose**: Define the pod with environment variables.

**YAML Content**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  containers:
  - name: print-env-container
    image: bash:latest
    imagePullPolicy: IfNotPresent
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "Nautilus"
    - name: GROUP
      value: "Ltd"
    command: ["/bin/sh", "-c", "echo $(GREETING) $(COMPANY) $(GROUP)"]
  restartPolicy: Never
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Pod`: Defines a pod resource.
- `metadata.name`: Sets pod name to `print-envars-greeting`.
- `env`: Defines three environment variables.
- `command`: Echoes variables in specified format.
- `restartPolicy: Never`: Ensures pod completes without restarting.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply the Pod Manifest

```bash
kubectl apply -f pod.yaml
```

**Purpose**: Deploy the pod.

**Expected Output**:
```
pod/print-envars-greeting created
```

---

## 🔹 Step 4: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm the pod is in `Completed` state.

**Expected Output**:
```
NAME                   READY   STATUS      RESTARTS   AGE
print-envars-greeting  0/1     Completed   0          10s
```

---

## 🔹 Step 5: Check Pod Logs

```bash
kubectl logs -f print-envars-greeting
```

**Purpose**: Verify the environment variables were echoed.

**Expected Output**:
```
Welcome to Nautilus Ltd
```

---

## 📋 Quick Command Reference

```bash
# Create and apply pod manifest
vi pod.yaml
# (Paste YAML content)
kubectl apply -f pod.yaml

# Verify pod and logs
kubectl get pods
kubectl logs -f print-envars-greeting
```

---

## 💡 Additional Tips

- **Task Variability**: Pod names or variable values may differ; adjust YAML as needed.
- **Completion Status**: `restartPolicy: Never` ensures one-time execution.
- **Logging**: Use `kubectl logs` for debugging completed pods.
- **Dry Run**: Test with `kubectl apply -f pod.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Completing**
**Symptoms**: Pod in `Error` or `CrashLoopBackOff`.
**Solution**: Check logs and command syntax.
```bash
kubectl describe pod print-envars-greeting
kubectl logs print-envars-greeting
```

### **Issue 2: Incorrect Output**
**Symptoms**: Logs show wrong or no output.
**Solution**: Verify environment variables and command.
```bash
kubectl describe pod print-envars-greeting
# Ensure env and command are correct
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod print-envars-greeting
# Ensure image: bash:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **configuring environment variables** and ensuring the pod completes with the correct output.

**💡 Solution Approach:**
1. **Pod Configuration**: Defined `bash:latest` with environment variables.
2. **Command Setup**: Used exact command to echo variables.
3. **Completion**: Set `restartPolicy: Never` for one-time execution.
4. **Verification**: Checked logs for correct output.

**🎯 Key Success Factors:**
- **Correct variables**: Set `GREETING`, `COMPANY`, `GROUP` as specified.
- **Command accuracy**: Used exact echo command.
- **Pod status**: Ensured `Completed` state.
- **Log verification**: Confirmed output `Welcome to Nautilus Ltd`.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `print-envars-greeting`.
- **Container Name**: Must be `print-env-container`.
- **Image**: Must be `bash:latest`.
- **Variables**: Must match specified values.
- **Command**: Must be exact.
- **Restart Policy**: Must be `Never`.

**🔒 Kubernetes Benefits:**
- **Environment Variables**: Simplifies configuration management.
- **One-Time Tasks**: Pods with `Never` policy suit simple scripts.
- **Debugging**: Logs provide clear output verification.

---

## ⚠️ Important Production Notes

🔧 **Task Execution**: Use Jobs for one-time tasks in production.

🔐 **Security**: Avoid hardcoding sensitive variables; use ConfigMaps or Secrets.

📊 **Monitoring**: Monitor pod completion and resource usage.

🛡️ **Optimization**: Use minimal images for simple tasks.

---