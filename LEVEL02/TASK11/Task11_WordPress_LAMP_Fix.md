# 🌟 Task 11 - Fix WordPress LAMP Stack Deployment on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team deployed a WordPress website on a LAMP stack within a Kubernetes cluster. The application was functioning correctly a few hours ago, but it has since gone down due to configuration issues. The deployment is named `lamp-wp`, and it uses a service named `lamp-service` with Apache running on the default HTTP port and a NodePort of `30008`. Logs indicate issues with database connectivity and other configuration problems. Environment variables such as `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, and `MYSQL_HOST` are associated with the pods. The task is to investigate and fix the issues to restore application functionality without modifying existing components like deployment name, service name, types, or labels.

### Requirements:
- Fix the `lamp-wp` deployment to ensure the WordPress application is accessible.
- Address database connectivity issues caused by incorrect environment variable references in the application code.
- Ensure the `lamp-service` uses NodePort `30008`.
- Verify the deployment is running with pods in a `Running` state and the application is accessible.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Do not delete or modify existing components like deployment name, service name, types, or labels.

👉 **Your task**: Investigate and fix the WordPress LAMP stack deployment to restore functionality.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

## 🔹 Step 2: Verify Deployment and Pods

```bash
kubectl get deployment
```

**Purpose**: Check the status of the `lamp-wp` deployment.

**Expected Output**:
```
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
lamp-wp  1/1     1            1           12m
```

**Verification**: Confirm the deployment shows `1/1` ready replicas.

```bash
kubectl get pods
```

**Purpose**: Ensure pods are in a `Running` state.

**Expected Output**:
```
NAME                        READY   STATUS    RESTARTS   AGE
lamp-wp-56c7c454fc-jjjjv   1/1     Running   0          12m
```

**Verification**: Confirm the pod is in `Running` state.

---

## 🔹 Step 3: Investigate Application Code

```bash
kubectl exec -it lamp-wp-56c7c454fc-jjjjv -c httpd-php-container -- sh
```

**Purpose**: Access the container to inspect the application code.

**Commands Inside Container**:
```bash
cd /app
ls
```

**Output**:
```
index.php
```

```bash
cat index.php
```

**Output**:
```php
<?php
$dbname = $_ENV['MYSQL_DATABASE'];
$dbuser = $_ENV['MYSQL_USER'];
$dbpass = $_ENV[''MYSQL_PASSWORD""];
$dbhost = $_ENV['MYSQL-HOST'];


$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
```

**Key Observation**:
- **Typo in Environment Variables**:
  - `$dbpass = $_ENV[''MYSQL_PASSWORD""]`: Incorrect syntax with double quotes and extra single quotes.
  - `$dbhost = $_ENV['MYSQL-HOST']`: Incorrect variable name (should be `MYSQL_HOST`).

**Verification**: The environment variables should be `MYSQL_PASSWORD` and `MYSQL_HOST` to match the defined variables.

---

## 🔹 Step 4: Fix Application Code

**Inside Container**:
```bash
vi /app/index.php
```

**Corrected Code**:
```php
<?php
$dbname = $_ENV['MYSQL_DATABASE'];
$dbuser = $_ENV['MYSQL_USER'];
$dbpass = $_ENV['MYSQL_PASSWORD'];
$dbhost = $_ENV['MYSQL_HOST'];

$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
```

**Changes Made**:
- Corrected `$dbpass = $_ENV[''MYSQL_PASSWORD""]` to `$dbpass = $_ENV['MYSQL_PASSWORD']`.
- Corrected `$dbhost = $_ENV['MYSQL-HOST']` to `$dbhost = $_ENV['MYSQL_HOST']`.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Restart Apache**:
```bash
service httpd restart
service httpd status
```

**Expected Output**:
```
apache:apached                   RUNNING   pid 355, uptime 0:00:00
```

**Verification**: Ensure Apache is running after restarting.

**Exit Container**:
```bash
exit
```

---

## 🔹 Step 5: Verify Service Configuration

```bash
kubectl get svc lamp-service
```

**Purpose**: Check the NodePort configuration of `lamp-service`.

**Output**:
```
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
lamp-service   NodePort   10.96.212.234   <none>        80:30009/TCP   12m
```

**Key Observation**:
- The service uses NodePort `30009`, but the requirement specifies `30008`.

---

## 🔹 Step 6: Fix Service NodePort

```bash
kubectl patch svc lamp-service -p '{"spec": {"ports": [{"port":80,"targetPort":80,"nodePort":30008}]}}'
```

**Purpose**: Update the NodePort to `30008`.

**Expected Output**:
```
service/lamp-service patched
```

**Verify**:
```bash
kubectl get svc lamp-service
```

**Expected Output**:
```
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
lamp-service   NodePort   10.96.212.234   <none>        80:30008/TCP   12m
```

**Verification**: Confirm the NodePort is `30008`.

---

## 🔹 Step 7: Verify Application Accessibility

**Access the Application**:
- click on APP icon given in top corner  
- Use the cluster's external IP or Node IP with port `30008` (e.g., via a browser or `curl`).
- Example: `curl http://<node-ip>:30008`

**Expected Output**:
```
Connected successfully
```

**Verification**: Confirm the application is accessible and displays the expected output.

---

## 📋 Quick Command Reference

```bash
# Verify deployment and pods
kubectl get deployment
kubectl get pods

# Investigate application code
kubectl exec -it lamp-wp-56c7c454fc-jjjjv -c httpd-php-container -- sh
cd /app
cat index.php
vi /app/index.php
service httpd restart
service httpd status
exit

# Verify and fix service
kubectl get svc lamp-service
kubectl patch svc lamp-service -p '{"spec": {"ports": [{"port":80,"targetPort":80,"nodePort":30008}]}}'
kubectl get svc lamp-service

# Test application
click on APP button given in lab
curl http://<node-ip>:30008
```

---

## 💡 Additional Tips

- **Task Variability**: Environment variable names or NodePort values may differ; always verify with `kubectl describe` or `kubectl get`.
- **Pod Logs**: Check logs for additional database connectivity issues.
  ```bash
  kubectl logs lamp-wp-56c7c454fc-jjjjv -c httpd-php-container
  ```
- **Dry Run**: Test service changes with:
  ```bash
  kubectl patch svc lamp-service -p '{"spec": {"ports": [{"port":80,"targetPort":80,"nodePort":30008}]}}' --dry-run=client
  ```
- **Database Connectivity**: Ensure the `MYSQL_HOST` points to the correct MySQL service and that credentials are valid.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pods Not Running**
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod lamp-wp-56c7c454fc-jjjjv
kubectl logs lamp-wp-56c7c454fc-jjjjv -c httpd-php-container
```

### **Issue 2: Database Connectivity Failure**
**Symptoms**: Application fails to connect to the database.
**Solution**: Verify environment variables and MySQL service accessibility.
```bash
kubectl exec -it lamp-wp-56c7c454fc-jjjjv -c httpd-php-container -- env | grep MYSQL
```

### **Issue 3: Incorrect NodePort**
**Symptoms**: Service uses wrong NodePort (`30009` instead of `30008`).
**Solution**: Patch the service as shown in Step 6.

### **Issue 4: Apache Not Reflecting Changes**
**Symptoms**: Application code changes not taking effect.
**Solution**: Restart Apache or redeploy the pod.
```bash
kubectl rollout restart deployment lamp-wp
```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Database Connectivity Issue**:
   - Typos in environment variable references in `/app/index.php`:
     - `$_ENV[''MYSQL_PASSWORD""]` instead of `$_ENV['MYSQL_PASSWORD']`.
     - `$_ENV['MYSQL-HOST']` instead of `$_ENV['MYSQL_HOST']`.
2. **Incorrect NodePort**:
   - Service `lamp-service` used NodePort `30009` instead of the required `30008`.

### 💡 Solution Approach:
1. **Fixed Application Code**:
   - Corrected typos in `index.php` to use proper environment variable names.
   - Restarted Apache to apply changes.
2. **Fixed Service Configuration**:
   - Patched `lamp-service` to use NodePort `30008`.
3. **Verified Fix**:
   - Ensured pods are running and the application is accessible at port `30008`.

### 🎯 Key Success Factors:
- **Correct Environment Variables**: Proper references to `MYSQL_PASSWORD` and `MYSQL_HOST` resolve database connectivity.
- **Correct NodePort**: NodePort `30008` ensures application accessibility.
- **Pod Status**: Pods in `Running` state.
- **Application Output**: Displays `Connected successfully`.

### ⚠️ Critical Configuration Details:
- **Deployment Name**: Must be `lamp-wp`.
- **Service Name**: Must be `lamp-service`.
- **NodePort**: Must be `30008`.
- **Environment Variables**: Must use `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`.

### 🔒 Kubernetes Benefits:
- **Reliability**: Correct configurations ensure stable WordPress deployment.
- **Accessibility**: NodePort service enables external access.
- **Debugging**: Kubernetes logs and `kubectl exec` provide clear diagnostics.

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.

🔐 **Security**: Secure MySQL with strong credentials and network policies.

📊 **Monitoring**: Monitor Apache and MySQL pod health and performance metrics.

🛡️ **Scalability**: Adjust replicas or use a MySQL cluster for high availability.