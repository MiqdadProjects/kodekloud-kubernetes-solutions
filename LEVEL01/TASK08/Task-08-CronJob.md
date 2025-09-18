**🌟 Task 8 - Create CronJob**

**📌 Task Description**

The Nautilus DevOps team needs to schedule regular tasks in Kubernetes. For now, they are creating a cron job with a dummy command to test scheduling functionality.

**Requirements:**
- Create a CronJob named `xfusion`.
- Set schedule to `*/9 * * * *` (every 9 minutes; adjustable).
- Use `nginx:latest` image with container name `cron-xfusion`.
- Run command `echo Welcome to xfusioncorp!`.
- Set restart policy to `OnFailure`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like job names or schedules may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Schedule a cron job to run a dummy command.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create CronJob Manifest

```bash
vi xfusion-cronjob.yaml
```

**Purpose**: Define the CronJob with specified schedule and command.

**YAML Content**:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: xfusion
spec:
  schedule: "*/9 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cron-xfusion
              image: nginx:latest
              imagePullPolicy: IfNotPresent
              command:
                - /bin/sh
                - -c
                - echo Welcome to xfusioncorp!
          restartPolicy: OnFailure
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: CronJob`: Defines a scheduled job resource.
- `spec.schedule`: Sets cron schedule `*/9 * * * *` (every 9 minutes).
- `jobTemplate`: Defines the job template.
- `spec.template.spec.containers`: Uses `nginx:latest` with command `echo Welcome to xfusioncorp!`.
- `restartPolicy: OnFailure`: Restarts only on job failure.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply the CronJob Manifest

```bash
kubectl apply -f xfusion-cronjob.yaml
```

**Purpose**: Deploy the CronJob.

**Expected Output**:
```
cronjob.batch/xfusion created
```

---

## 🔹 Step 4: Verify CronJob

```bash
kubectl get cronjob
```

**Purpose**: Confirm the CronJob is created.

**Expected Output**:
```
NAME      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
xfusion   */9 * * * *   False     0        <none>          10s
```

---

## 🔹 Step 5: Verify Job Execution

```bash
kubectl get jobs
```

**Purpose**: Check for jobs created by the CronJob.

**Expected Output** (after 9 minutes):
```
NAME               COMPLETIONS   DURATION   AGE
xfusion-xxx   1/1           5s         9m
```

**Verify Pods**:
```bash
kubectl get pods -l job-name=xfusion-xxx
```

**Expected Output**:
```
NAME                     READY   STATUS      RESTARTS   AGE
xfusion-xxx-xxx   0/1     Completed   0          9m
```

---

## 🔹 Step 6: Check Job Logs

```bash
kubectl logs xfusion-xxx-xxx
```

**Purpose**: Verify the dummy command output.

**Expected Output**:
```
Welcome to xfusioncorp!
```

---

## 📋 Quick Command Reference

```bash
# Create and apply CronJob manifest
vi xfusion-cronjob.yaml
# (Paste YAML content)
kubectl apply -f xfusion-cronjob.yaml

# Verify CronJob and jobs
kubectl get cronjob
kubectl get jobs
kubectl get pods -l job-name=xfusion-xxx
kubectl logs xfusion-xxx-xxx
```

---

## 💡 Additional Tips

- **Task Variability**: CronJob names or schedules may differ; adjust YAML as needed.
- **Schedule Testing**: Test cron schedule with `kubectl create job --from=cronjob/xfusion test-job`.
- **Logging**: Ensure job output is logged for debugging.
- **Dry Run**: Test with `kubectl apply -f xfusion-cronjob.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Job Not Running**
**Symptoms**: No jobs created by CronJob.
**Solution**: Check CronJob schedule and status.
```bash
kubectl describe cronjob xfusion
# Verify schedule and suspend status
```

### **Issue 2: Pod Failed**
**Symptoms**: Job pod in `Error` state.
**Solution**: Check pod logs.
```bash
kubectl logs xfusion-xxx-xxx
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image.
```bash
kubectl describe pod xfusion-xxx-xxx
# Ensure image: nginx:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **configuring a CronJob** with a specific schedule and dummy command while ensuring correct execution.

**💡 Solution Approach:**
1. **CronJob Configuration**: Set schedule to `*/9 * * * *` and command to `echo`.
2. **Image and Container**: Used `nginx:latest` with `cron-xfusion` container name.
3. **Verification**: Checked job and pod status with logs.
4. **Best Practices**: Set `restartPolicy: OnFailure` for job reliability.

**🎯 Key Success Factors:**
- **Correct schedule**: `*/9 * * * *` as required.
- **Command execution**: `echo Welcome to xfusioncorp!` verified.
- **Job status**: Ensured job completes successfully.
- **Minimal YAML**: Clear and concise configuration.

**⚠️ Critical Configuration Details:**
- **CronJob Name**: Must be `xfusion`.
- **Image**: Must be `nginx:latest`.
- **Schedule**: Must be `*/9 * * * *` (or as specified).
- **Command**: Must output `Welcome to xfusioncorp!`.

**🔒 Kubernetes Benefits:**
- **Automation**: Schedules recurring tasks.
- **Reliability**: `OnFailure` restart policy ensures retries.
- **Logging**: Captures job output for debugging.

---

## ⚠️ Important Production Notes

🔧 **CronJob Scheduling**: Automates recurring tasks efficiently.

🔐 **Security**: Use minimal images and secure commands in production.

📊 **Monitoring**: Monitor job execution and logs.

🛡️ **Resource Control**: Set resource limits for jobs in production.

---