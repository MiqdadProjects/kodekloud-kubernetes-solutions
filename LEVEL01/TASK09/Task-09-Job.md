**🌟 Task 9 - Create Countdown Job**

**📌 Task Description**

The Nautilus DevOps team is preparing templates for Kubernetes jobs to execute one-time tasks. A team member has been tasked with creating a job to test a simple command execution, ensuring it completes successfully.

**Requirements:**
- Create a job named `countdown-nautilus`.
- Set template metadata name to `countdown-nautilus`.
- Use container name `container-countdown-nautilus` with `debian:latest` image (specify the tag).
- Run command `sleep 5`.
- Set restart policy to `Never`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like job names, images, or commands may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Create a job to execute a sleep command and ensure it completes.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host where `kubectl` is configured to interact with the Kubernetes cluster.

---

## 🔹 Step 2: Create Job Manifest

```bash
vi countdown-nautilus.yaml
```

**Purpose**: Define the job with the specified command and image.

**YAML Content**:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-nautilus
spec:
  template:
    metadata:
      name: countdown-nautilus
    spec:
      containers:
        - name: container-countdown-nautilus
          image: debian:latest
          imagePullPolicy: IfNotPresent
          command: ["/bin/sh", "-c", "sleep 5"]
      restartPolicy: Never
  backoffLimit: 4
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Job`: Defines a one-time job resource.
- `metadata.name`: Sets job name to `countdown-nautilus`.
- `spec.template.metadata.name`: Sets template name to `countdown-nautilus`.
- `spec.template.spec.containers`: Uses `debian:latest` with `sleep 5` command.
- `restartPolicy: Never`: Ensures no restarts on completion.
- `backoffLimit: 4`: Allows 4 retry attempts on failure.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply the Job Manifest

```bash
kubectl apply -f countdown-nautilus.yaml
```

**Purpose**: Deploy the job to the Kubernetes cluster.

**Expected Output**:
```
job.batch/countdown-nautilus created
```

---

## 🔹 Step 4: Verify Job

```bash
kubectl get jobs
```

**Purpose**: Confirm the job is created and completed.

**Expected Output**:
```
NAME                COMPLETIONS   DURATION   AGE
countdown-nautilus  1/1           5s         10s
```

---

## 🔹 Step 5: Verify Pod

```bash
kubectl get pods -l job-name=countdown-nautilus
```

**Purpose**: Ensure the job pod completed successfully.

**Expected Output**:
```
NAME                      READY   STATUS      RESTARTS   AGE
countdown-nautilus-xxx   0/1     Completed   0          10s
```

---

## 🔹 Step 6: Check Job Logs

```bash
kubectl logs countdown-nautilus-xxx
```

**Purpose**: Verify the command executed (no output expected for `sleep 5`, but confirms successful execution).

**Expected Output**:
```
<no output>
```

---

## 📋 Quick Command Reference

```bash
# Create and apply job manifest
vi countdown-nautilus.yaml
# (Paste YAML content)
kubectl apply -f countdown-nautilus.yaml

# Verify job and pod
kubectl get jobs
kubectl get pods -l job-name=countdown-nautilus
kubectl logs countdown-nautilus-xxx
```

---

## 💡 Additional Tips

- **Task Variability**: Job names, images, or commands may differ; adjust the YAML as needed.
- **Job Completion**: Ensure the job reaches `Completed` status for success.
- **Logging**: Add logging commands (e.g., `echo`) for debugging in production.
- **Dry Run**: Test with `kubectl apply -f countdown-nautilus.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Job Not Completing**
**Symptoms**: Job stuck or in `Error` state.
**Solution**: Check pod events and logs.
```bash
kubectl describe job countdown-nautilus
kubectl describe pod countdown-nautilus-xxx
kubectl logs countdown-nautilus-xxx
```

### **Issue 2: Image Pull Failure**
**Symptoms**: Pod in `ErrImagePull` or `ImagePullBackOff` state.
**Solution**: Verify image name and connectivity.
```bash
kubectl describe pod countdown-nautilus-xxx
# Ensure image: debian:latest
```

### **Issue 3: Pod Failed**
**Symptoms**: Pod in `Error` state.
**Solution**: Check command syntax and resource availability.
```bash
kubectl describe pod countdown-nautilus-xxx
# Verify command: ["/bin/sh", "-c", "sleep 5"]
```

### **Issue 4: Job Not Found**
**Symptoms**: `kubectl get jobs` returns no results.
**Solution**: Ensure the job was applied correctly.
```bash
kubectl apply -f countdown-nautilus.yaml
kubectl get jobs
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **creating a one-time job** with a simple `sleep 5` command, ensuring correct naming, image, and completion status.

**💡 Solution Approach:**
1. **Job Configuration**: Defined the job with `debian:latest` and `sleep 5` command.
2. **Naming Accuracy**: Set job and template metadata names to `countdown-nautilus`.
3. **Verification**: Confirmed job completion and pod status with `kubectl get` commands.
4. **Best Practices**: Used `restartPolicy: Never` for one-time execution and `backoffLimit: 4` for retry handling.

**🎯 Key Success Factors:**
- **Correct naming**: Job and template named `countdown-nautilus`.
- **Command execution**: `sleep 5` completed successfully.
- **Pod status**: Ensured `Completed` state.
- **Minimal YAML**: Kept configuration concise and clear.

**⚠️ Critical Configuration Details:**
- **Job Name**: Must be `countdown-nautilus`.
- **Template Metadata Name**: Must be `countdown-nautilus`.
- **Container Name**: Must be `container-countdown-nautilus`.
- **Image**: Must be `debian:latest`.
- **Command**: Must be `sleep 5`.
- **Restart Policy**: Must be `Never`.

**🔒 Kubernetes Benefits:**
- **One-Time Execution**: Jobs ensure tasks run to completion.
- **Reliability**: `backoffLimit` allows retries for transient failures.
- **Simplicity**: Minimal configuration for simple tasks.

---

## ⚠️ Important Production Notes

🔧 **Job Execution**: Ideal for one-time or batch tasks in production.

🔐 **Security**: Use minimal images (e.g., `debian:slim`) and secure commands.

📊 **Monitoring**: Monitor job completion and pod resource usage.

🛡️ **Scalability**: Use parallel jobs for large-scale tasks if needed.

---