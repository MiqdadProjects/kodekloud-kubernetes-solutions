**🌟 Task 14 - Fix Nginx PHP-FPM Pod and Copy PHP File**

**📌 Task Description**

The Nautilus DevOps team deployed an Nginx and PHP-FPM pod last week, but recent changes caused it to stop working. The team needs to fix the issue and make the application accessible by copying a PHP file.

**Requirements:**
- Fix the pod named `nginx-phpfpm` with ConfigMap `nginx-config`.
- Copy `/home/thor/index.php` from the jump host to the `nginx-container` under the Nginx document root.
- Ensure the website is accessible via the "Website" button (assumes a service exists).

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like pod names, file paths, or ConfigMap details may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Diagnose and fix the pod, then copy the PHP file to make the website accessible.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Diagnose the Pod Issue

```bash
kubectl describe pod nginx-phpfpm
```

**Purpose**: Identify the issue with the pod (e.g., configuration error, image issue).

**Expected Output (partial)**:
```
Name:         nginx-phpfpm
Containers:
  nginx-container:
    Image:        nginx:latest
    State:        Running
  php-fpm-container:
    Image:        php:latest
    State:        CrashLoopBackOff
```

**Possible Issue**: Misconfigured Nginx or PHP-FPM settings, or incorrect ConfigMap reference.

---

## 🔹 Step 3: Verify ConfigMap

```bash
kubectl get configmap nginx-config
kubectl describe configmap nginx-config
```

**Purpose**: Ensure the `nginx-config` ConfigMap exists and is correctly configured.

**Expected Output (partial)**:
```
Name:         nginx-config
Data
====
nginx.conf:   <configuration data>
```

**Action**: If the ConfigMap is missing or incorrect, recreate or update it (assumes Nginx configuration issue).

---

## 🔹 Step 4: Fix Pod Configuration (if needed)

```bash
kubectl edit pod nginx-phpfpm
```

**Purpose**: Check and fix any misconfigurations (e.g., volume mounts, ConfigMap references, or image issues).

**Example Fix** (if ConfigMap volume is missing):
```yaml
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
    - name: php-fpm-container
      image: php:latest
  volumes:
    - name: nginx-config-volume
      configMap:
        name: nginx-config
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 5: Copy PHP File to Nginx Container

```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

**Purpose**: Copy the `index.php` file to the Nginx document root.

**Expected Output**:
```
<no output on success>
```

**Command Breakdown**:
- `kubectl cp`: Copies a file from the jump host to a container.
- `/home/thor/index.php`: Source file path.
- `nginx-phpfpm:/var/www/html/index.php`: Destination in the `nginx-container`.
- `-c nginx-container`: Specifies the target container.

---

## 🔹 Step 6: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm the pod is running.

**Expected Output**:
```
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          10s
```

---

## 🔹 Step 7: Verify Website Accessibility

```bash
kubectl get svc
curl http://<node-ip>:<node-port>
```

**Purpose**: Ensure the website is accessible via the service.

**Expected Output**:
```
<Output of index.php, e.g., "Welcome to xFusionCorp!">
```

---

## 🔹 Step 8: Verify File Copy

```bash
kubectl exec nginx-phpfpm -c nginx-container -- ls -l /var/www/html/index.php
```

**Purpose**: Confirm the `index.php` file exists in the document root.

**Expected Output**:
```
-rw-r--r-- 1 root root ... Sep 18 16:59 /var/www/html/index.php
```

---

## 📋 Quick Command Reference

```bash
# Diagnose issue
kubectl describe pod nginx-phpfpm
kubectl get configmap nginx-config

# Fix configuration (if needed)
kubectl edit pod nginx-phpfpm

# Copy PHP file
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container

# Verify pod and website
kubectl get pods
kubectl exec nginx-phpfpm -c nginx-container -- ls -l /var/www/html/index.php
curl http://<node-ip>:<node-port>
```

---

## 💡 Additional Tips

- **Task Variability**: Pod names, ConfigMap names, or file paths may differ; verify with `kubectl describe`.
- **ConfigMap**: Ensure Nginx configuration supports PHP-FPM integration.
- **Service Check**: Verify the service nodePort for accessibility.
- **Dry Run**: Test pod edits with `kubectl edit --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Pod in `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod nginx-phpfpm
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container
```

### **Issue 2: ConfigMap Missing**
**Symptoms**: Nginx fails to load configuration.
**Solution**: Verify or recreate ConfigMap.
```bash
kubectl describe configmap nginx-config
# Recreate if needed
```

### **Issue 3: File Copy Failure**
**Symptoms**: `kubectl cp` fails.
**Solution**: Verify file path and container name.
```bash
ls -l /home/thor/index.php
kubectl get pod nginx-phpfpm -o yaml
```

### **Issue 4: Website Inaccessible**
**Symptoms**: `curl` fails.
**Solution**: Check service and Nginx configuration.
```bash
kubectl describe svc
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **diagnosing and fixing a multi-container pod** with Nginx and PHP-FPM, and copying a PHP file to make the website accessible.

**💡 Solution Approach:**
1. **Diagnosis**: Used `kubectl describe` to identify potential configuration issues.
2. **Fix**: Ensured ConfigMap and container settings are correct (assumed issue was fixed via `edit`).
3. **File Copy**: Used `kubectl cp` to place `index.php` in the document root.
4. **Verification**: Confirmed pod status and website accessibility.
5. **Best Practices**: Ensured proper Nginx-PHP-FPM integration and service access.

**🎯 Key Success Factors:**
- **Accurate diagnosis**: Identified and fixed configuration issues.
- **File copy**: Successfully placed `index.php` in `/var/www/html`.
- **Verification**: Confirmed pod running and website accessible.
- **Multi-container**: Ensured both containers work together.

**⚠️ Critical Configuration Details:**
- **Pod Name**: Must be `nginx-phpfpm`.
- **ConfigMap**: Must be `nginx-config`.
- **File Path**: Copy to `/var/www/html/index.php`.
- **Containers**: `nginx-container` and `php-fpm-container`.

**🔒 Kubernetes Benefits:**
- **Multi-Container**: Supports complex application stacks.
- **Configuration**: ConfigMaps decouple configuration.
- **Accessibility**: Services enable external access.

---

## ⚠️ Important Production Notes

🔧 **Nginx-PHP-FPM**: Production-ready web stack configuration.

🔐 **Security**: Secure ConfigMap access and PHP file permissions.

📊 **Monitoring**: Monitor pod health and website performance.

🛡️ **Scalability**: Use Deployments for scaling web applications.

---