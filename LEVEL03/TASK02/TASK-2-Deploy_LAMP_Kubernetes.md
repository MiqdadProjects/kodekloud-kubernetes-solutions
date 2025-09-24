# 🌟 Task 2 - Deploy PHP Website with Apache and MySQL on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team aims to deploy a PHP website on a Kubernetes cluster, utilizing Apache as the web server and MySQL as the database. The team has gathered all requirements and is ready to make the website live. This task involves creating the necessary Kubernetes resources to ensure the application is functional and accessible.

### Requirements:
- Create a ConfigMap named **`php-config`** for `php.ini` with `variables_order = "EGPCS"`.
- Create a deployment named **`lamp-wp`** with:
  - A container named **`httpd-php-container`** using the image `webdevops/php-apache:alpine-3-php7`.
  - A container named **`mysql-container`** using the image `mysql:5.6`.
  - Mount the `php-config` ConfigMap in the `httpd-php-container` at `/opt/docker/etc/php/php.ini`.
- Create a Kubernetes generic secret for MySQL-related values (`mysql-root-password`, `mysql-database`, `mysql-user`, `mysql-password`, `mysql-host`) with custom values.
- Add environment variables (`MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`) to both containers, sourced from the secret using the `env` field.
- Create a NodePort service named **`lamp-service`** to expose the web application on **NodePort 30008**.
- Create a ClusterIP service named **`mysql-service`** for MySQL on port **3306**.
- Copy the `/tmp/index.php` file from the jump host to the `httpd-php-container` under `/app`, ensuring MySQL-related variables use environment variables (not hardcoded).
- Ensure the application is accessible on NodePort 30008, displaying **"Connected successfully"**.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. 

👉 **Your task**: Deploy the PHP website with Apache and MySQL on the Kubernetes cluster, ensuring all components are correctly configured and the application is accessible.

Note: The `/tmp/index.php` file exists on the jump host and according to task we need to mofify it , and if permission issues arise, use `sudo chmod 777 /tmp/index.php` with the password `mjolnir123`.
---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- ConfigMap for PHP configuration
- Secret for MySQL credentials
- Deployment with Apache and MySQL containers
- NodePort service for web access
- ClusterIP service for MySQL database
- PHP application file (`index.php`)

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **ConfigMap**: Stores `php.ini` configuration for Apache
- **Secret**: Secures MySQL credentials
- **Deployment**: Runs Apache and MySQL containers
- **Services**: Exposes Apache on NodePort and MySQL internally
- **Application**: PHP website connects to MySQL and displays status

### 🎯 Implementation Strategy
1. Create a ConfigMap for `php.ini`
2. Create a secret for MySQL credentials
3. Deploy Apache and MySQL containers with environment variables
4. Expose the application via NodePort and MySQL via ClusterIP
5. Copy and configure `index.php` in the Apache container
6. Verify deployment, services, and application accessibility

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands. Use password `mjolnir123` if prompted.

---

### Step 2: Create ConfigMap
Create a file named `php-config.yaml`:

```yaml
# php-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
data:
  php.ini: |
    variables_order = "EGPCS"
```

**Purpose:** Define the PHP configuration for the Apache container.

---

### Step 3: Create Secret for MySQL
Create a file named `mysql-secrets.yaml`:

```yaml
# mysql-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets
type: Opaque
data:
  mysql-root-password: cm9vdHBhc3M=     # base64("rootpass")
  mysql-database: d2Vic2l0ZQ==         # base64("website")
  mysql-user: dXNlcg==                 # base64("user")
  mysql-password: dXNlcnBhc3M=         # base64("userpass")
  mysql-host: bXlzcWwtc2VydmljZQ==     # base64("mysql-service")
```

**Purpose:** Store MySQL credentials securely using base64-encoded values.

---

### Step 4: Create Deployment
Create a file named `lamp-deployment.yaml`:

```yaml
# lamp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lamp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp-app
  template:
    metadata:
      labels:
        app: lamp-app
    spec:
      containers:
        - name: httpd-php-container
          image: webdevops/php-apache:alpine-3-php7
          ports:
            - containerPort: 80
          volumeMounts:
            - name: php-config-volume
              mountPath: /opt/docker/etc/php/php.ini
              subPath: php.ini
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-root-password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-host
        - name: mysql-container
          image: mysql:5.6
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-root-password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-host
      volumes:
        - name: php-config-volume
          configMap:
            name: php-config
```

**Configuration Breakdown:**
- **Deployment**:
  - `name: lamp-wp` - Specifies the deployment name
  - `replicas: 1` - Runs one pod (as per solution)
  - `selector` and `labels` - Matches pods using `app: lamp-app`
- **httpd-php-container**:
  - `image: webdevops/php-apache:alpine-3-php7` - Apache with PHP
  - Mounts `php-config` at `/opt/docker/etc/php/php.ini`
  - Uses environment variables from `mysql-secrets`
- **mysql-container**:
  - `image: mysql:5.6` - MySQL database
  - Uses environment variables from `mysql-secrets`
- **Volume**:
  - Mounts `php-config` ConfigMap as a volume

---

### Step 5: Create Services
Create a file named `lamp-services.yaml`:

```yaml
# lamp-services.yaml
apiVersion: v1
kind: Service
metadata:
  name: lamp-service
spec:
  type: NodePort
  selector:
    app: lamp-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  type: ClusterIP
  selector:
    app: lamp-app
  ports:
    - port: 3306
      targetPort: 3306
```

**Configuration Breakdown:**
- **lamp-service**:
  - `type: NodePort` - Exposes Apache externally
  - `nodePort: 30008` - As required
  - `selector: app: lamp-app` - Links to deployment pods
- **mysql-service**:
  - `type: ClusterIP` - Internal MySQL access
  - `port: 3306` - MySQL default port
  - `selector: app: lamp-app` - Links to deployment pods

---

### Step 6: Modify and Copy index.php
If permission issues occur, fix them:
```bash
sudo chmod 777 /tmp/index.php
```

**Password:** `mjolnir123`

Modify `/tmp/index.php` to use environment variables:

```php
# /tmp/index.php
<?php
$dbname = getenv('MYSQL_DATABASE');
$dbuser = getenv('MYSQL_USER');
$dbpass = getenv('MYSQL_PASSWORD');
$dbhost = getenv('MYSQL_HOST');

$connect = mysqli_connect($dbhost, $dbuser, $dbpass, $dbname);

if (!$connect) {
   die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
?>
```

**Purpose:** Ensure `index.php` uses environment variables for MySQL connection details.

---

### Step 7: Apply Configurations
```bash
kubectl apply -f php-config.yaml
kubectl apply -f mysql-secrets.yaml
kubectl apply -f lamp-deployment.yaml
kubectl apply -f lamp-services.yaml
```

**Expected Output:**
```
configmap/php-config created
secret/mysql-secrets created
deployment.apps/lamp-wp created
service/lamp-service created
service/mysql-service created
```

---

### Step 8: Copy index.php to Apache Container
```bash
POD=$(kubectl get pod -l app=lamp-app -o jsonpath='{.items[0].metadata.name}')
kubectl cp /tmp/index.php $POD:/app -c httpd-php-container
```

**Explanation of Commands:**
- **Command 1: `POD=$(kubectl get pod -l app=lamp-app -o jsonpath='{.items[0].metadata.name}')`**  
  - **Purpose**: Retrieves the name of the first pod with the label `app=lamp-app` and stores it in the `POD` variable.  
  - **Details**:  
    - `kubectl get pod`: Lists pods in the current namespace.  
    - `-l app=lamp-app`: Filters pods with the label `app=lamp-app`, matching the deployment's selector.  
    - `-o jsonpath='{.items[0].metadata.name}'`: Extracts the name of the first pod using JSONPath.  
    - `POD=$()`: Stores the pod name (e.g., `lamp-wp-5f7b8d4c5-xyz12`) in the `POD` variable.  
  - **Why Needed**: Pod names are dynamically generated by Kubernetes, so this command dynamically retrieves the correct pod name for the file copy operation.

- **Command 2: `kubectl cp /tmp/index.php $POD:/app -c httpd-php-container`**  
  - **Purpose**: Copies the `index.php` file from the jump host's `/tmp` directory to the `/app` directory in the `httpd-php-container` of the specified pod.  
  - **Details**:  
    - `kubectl cp`: Copies files between the local filesystem and a container.  
    - `/tmp/index.php`: Source file on the jump host.  
    - `$POD:/app`: Destination path, where `$POD` is the pod name and `/app` is the Apache document root.  
    - `-c httpd-php-container`: Specifies the target container within the pod (since the pod has both `httpd-php-container` and `mysql-container`).  
  - **Why Needed**: The `index.php` file contains the PHP code for the website, which must be placed in the Apache document root to make the application functional.

---

### Step 9: Verify Deployment and Services
```bash
kubectl get configmap php-config
kubectl get secret mysql-secrets
kubectl get deploy lamp-wp
kubectl get pods -l app=lamp-app
kubectl get svc
```

**Expected Outputs:**
- ConfigMap:
  ```
  NAME         DATA   AGE
  php-config   1      2m
  ```
- Secret:
  ```
  NAME           TYPE     DATA   AGE
  mysql-secrets  Opaque   5      2m
  ```
- Deployment:
  ```
  NAME      READY   UP-TO-DATE   AVAILABLE   AGE
  lamp-wp   1/1     1            1           2m
  ```
- Pods:
  ```
  NAME                      READY   STATUS    RESTARTS   AGE
  lamp-wp-5f7b8d4c5-xyz12  2/2     Running   0          2m
  ```
- Services:
  ```
  NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
  lamp-service    NodePort    10.96.123.456   <none>        80:30008/TCP     2m
  mysql-service   ClusterIP   10.96.123.457   <none>        3306/TCP          2m
  ```

---

### Step 10: Verify Application Accessibility
**Access the Application**:
- Click on the **App** button in the lab interface.
- Alternatively, use the cluster's external IP or Node IP with port `30008`:
  ```bash
  curl http://<node-ip>:30008
  ```

**Expected Output:**
```
Connected successfully
```

**Verification:** Confirm the application displays "Connected successfully".

---

## 🔍 Code Analysis

### ConfigMap (`php-config.yaml`)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
data:
  php.ini: |
    variables_order = "EGPCS"
```

### Secret (`mysql-secrets.yaml`)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets
type: Opaque
data:
  mysql-root-password: cm9vdHBhc3M=     # base64("rootpass")
  mysql-database: d2Vic2l0ZQ==         # base64("website")
  mysql-user: dXNlcg==                 # base64("user")
  mysql-password: dXNlcnBhc3M=         # base64("userpass")
  mysql-host: bXlzcWwtc2VydmljZQ==     # base64("mysql-service")
```

### Deployment (`lamp-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lamp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp-app
  template:
    metadata:
      labels:
        app: lamp-app
    spec:
      containers:
        - name: httpd-php-container
          image: webdevops/php-apache:alpine-3-php7
          ports:
            - containerPort: 80
          volumeMounts:
            - name: php-config-volume
              mountPath: /opt/docker/etc/php/php.ini
              subPath: php.ini
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-root-password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-host
        - name: mysql-container
          image: mysql:5.6
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-root-password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: mysql-host
      volumes:
        - name: php-config-volume
          configMap:
            name: php-config
```

### Services (`lamp-services.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: lamp-service
spec:
  type: NodePort
  selector:
    app: lamp-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  type: ClusterIP
  selector:
    app: lamp-app
  ports:
    - port: 3306
      targetPort: 3306
```

### Application File (`/tmp/index.php`)
```php
<?php
$dbname = getenv('MYSQL_DATABASE');
$dbuser = getenv('MYSQL_USER');
$dbpass = getenv('MYSQL_PASSWORD');
$dbhost = getenv('MYSQL_HOST');

$connect = mysqli_connect($dbhost, $dbuser, $dbpass, $dbname);

if (!$connect) {
   die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
?>
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **ConfigMap** | name | php-config | Stores `php.ini` configuration |
| **Secret** | name | mysql-secrets | Stores MySQL credentials |
| **Deployment** | name | lamp-wp | Runs Apache and MySQL containers |
| **Deployment** | replicas | 1 | Single pod with two containers |
| **httpd-php-container** | image | webdevops/php-apache:alpine-3-php7 | Apache with PHP |
| **mysql-container** | image | mysql:5.6 | MySQL database |
| **lamp-service** | type | NodePort | Exposes Apache on port 30008 |
| **mysql-service** | type | ClusterIP | Exposes MySQL internally on port 3306 |

---

## ✅ Verification Steps

### Step 1: Verify ConfigMap
```bash
kubectl get configmap php-config
```

**Expected Output:**
```
NAME         DATA   AGE
php-config   1      2m
```

### Step 2: Verify Secret
```bash
kubectl get secret mysql-secrets
```

**Expected Output:**
```
NAME           TYPE     DATA   AGE
mysql-secrets  Opaque   5      2m
```

### Step 3: Verify Deployment
```bash
kubectl get deploy lamp-wp
```

**Expected Output:**
```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
lamp-wp   1/1     1            1           2m
```

### Step 4: Verify Pods
```bash
kubectl get pods -l app=lamp-app
```

**Expected Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE
lamp-wp-5f7b8d4c5-xyz12  2/2     Running   0          2m
```

### Step 5: Verify Services
```bash
kubectl get svc
```

**Expected Output:**
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
lamp-service    NodePort    10.96.123.456   <none>        80:30008/TCP     2m
mysql-service   ClusterIP   10.96.123.457   <none>        3306/TCP          2m
```

### Step 6: Test Application
```bash
curl http://<node-ip>:30008
```

**Expected Output:**
```
Connected successfully
```

---

## 🧪 Testing

### Verify ConfigMap Mount
```bash
POD=$(kubectl get pod -l app=lamp-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -c httpd-php-container -- cat /opt/docker/etc/php/php.ini
```

**Expected Output:**
```
variables_order = "EGPCS"
```

### Verify Environment Variables
```bash
kubectl exec $POD -c httpd-php-container -- env | grep MYSQL
```

**Expected Output:**
```
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=website
MYSQL_USER=user
MYSQL_PASSWORD=userpass
MYSQL_HOST=mysql-service
```

### Verify MySQL Service
```bash
kubectl describe svc mysql-service
```

**Purpose:** Confirm MySQL service is correctly configured.

---

## 📚 Quick Command Reference

```bash
# Fix permissions for index.php (if needed)
sudo chmod 777 /tmp/index.php  # Password: mjolnir123

# Apply configurations
kubectl apply -f php-config.yaml
kubectl apply -f mysql-secrets.yaml
kubectl apply -f lamp-deployment.yaml
kubectl apply -f lamp-services.yaml

# Copy index.php to Apache container
POD=$(kubectl get pod -l app=lamp-app -o jsonpath='{.items[0].metadata.name}')
kubectl cp /tmp/index.php $POD:/app -c httpd-php-container

# Verify resources
kubectl get configmap php-config
kubectl get secret mysql-secrets
kubectl get deploy lamp-wp
kubectl get pods -l app=lamp-app
kubectl get svc

# Test application
curl http://<node-ip>:30008
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms:** Pods in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod -l app=lamp-app
kubectl logs -l app=lamp-app -c httpd-php-container
kubectl logs -l app=lamp-app -c mysql-container
```

### Issue 2: Database Connectivity Failure
**Symptoms:** Application fails to connect to MySQL.
**Solution:** Verify environment variables and MySQL service.
```bash
kubectl exec $POD -c httpd-php-container -- env | grep MYSQL
kubectl describe svc mysql-service
```

### Issue 3: Incorrect NodePort
**Symptoms:** `lamp-service` uses wrong NodePort.
**Solution:** Patch the service.
```bash
kubectl patch svc lamp-service -p '{"spec": {"ports": [{"port":80,"targetPort":80,"nodePort":30008}]}}'
```

### Issue 4: ConfigMap Not Mounted
**Symptoms:** PHP configuration not applied.
**Solution:** Verify ConfigMap mount.
```bash
kubectl exec $POD -c httpd-php-container -- cat /opt/docker/etc/php/php.ini
```

### Issue 5: File Copy Failure
**Symptoms:** `kubectl cp` fails or `index.php` not found in container.
**Solution:** Verify pod name, container, and file permissions.
```bash
kubectl get pod -l app=lamp-app
kubectl exec $POD -c httpd-php-container -- ls /app
sudo chmod 777 /tmp/index.php  # Password: mjolnir123
```

---

## 💡 Additional Tips

- **Pod Logs**: Check logs for runtime issues.
  ```bash
  kubectl logs -l app=lamp-app -c httpd-php-container
  kubectl logs -l app=lamp-app -c mysql-container
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f <file>.yaml --dry-run=client
  ```
- **MySQL Health**: Ensure MySQL container initializes correctly.
  ```bash
  kubectl exec $POD -c mysql-container -- mysqladmin -u user -puserpass status
  ```
- **NodePort Range**: Verify NodePort 30008 is within the cluster’s allowed range (30000-32767).

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **ConfigMap Mounting**: Correctly mounting `php.ini` in the Apache container.
2. **Secret Management**: Securely providing MySQL credentials via secrets.
3. **Environment Variables**: Using `env` field to source variables from secrets.
4. **Service Configuration**: Ensuring correct NodePort and ClusterIP settings.
5. **File Copy**: Dynamically copying `index.php` to the correct Apache container.

### 💡 Solution Approach:
1. **ConfigMap**: Created `php-config` for `php.ini`.
2. **Secret**: Defined `mysql-secrets` with base64-encoded MySQL credentials.
3. **Deployment**: Configured `lamp-wp` with Apache and MySQL containers.
4. **Services**: Set up `lamp-service` (NodePort 30008) and `mysql-service` (ClusterIP 3306).
5. **Application**: Modified and copied `index.php` to use environment variables.
6. **Verification**: Ensured all components are running and the application is accessible.

### 🎯 Key Success Factors:
- **ConfigMap**: Mounted correctly at `/opt/docker/etc/php/php.ini`
- **Secret**: Provides secure MySQL credentials
- **Environment Variables**: Sourced from secrets using `env` field
- **Services**: Correctly expose Apache and MySQL
- **Application Output**: Displays "Connected successfully"

### ⚠️ Critical Configuration Details:
- **Deployment Name**: `lamp-wp`
- **Service Names**: `lamp-service`, `mysql-service`
- **NodePort**: 30008
- **MySQL Port**: 3306
- **Images**: `webdevops/php-apache:alpine-3-php7`, `mysql:5.6`
- **Environment Variables**: `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`

### 🔒 Kubernetes Benefits:
- **Scalability**: Deployment supports multiple replicas
- **Security**: Secrets secure sensitive data
- **Accessibility**: NodePort enables external access
- **Reliability**: Kubernetes manages pod health and restarts

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.

🔐 **Security**: Implement network policies to restrict MySQL access.

📊 **Monitoring**: Monitor Apache and MySQL pod health and performance.

🛡️ **Scalability**: Use separate MySQL deployment for production-grade setups.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **ConfigMap**: Managing configuration files
- **Secret**: Securing sensitive data
- **Multi-Container Pods**: Running Apache and MySQL together
- **Services**: Exposing applications and databases
- **Environment Variables**: Sourcing from secrets
- **File Management**: Copying files to containers with `kubectl cp`

**Kubernetes Features Used:**
- `ConfigMap` for PHP configuration
- `Secret` for MySQL credentials
- `apps/v1 Deployment` for multi-container pods
- `v1 Service` for NodePort and ClusterIP
- `kubectl cp` for file transfer

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **ConfigMap** - Created `php-config` for `php.ini`
- **Secret** - Created `mysql-secrets` for MySQL credentials
- **Deployment** - Created `lamp-wp` with Apache and MySQL containers
- **Services** - Created `lamp-service` (NodePort 30008) and `mysql-service` (ClusterIP 3306)
- **Application** - Copied and configured `index.php` to use environment variables
- **Verification** - Confirmed all components and application accessibility

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The PHP website with Apache and MySQL is now deployed on the Kubernetes cluster, accessible via NodePort 30008, and displays "Connected successfully", ready for further enhancements by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Scaling**: Increase replicas or use autoscaling
- **Security**: Add network policies and secret encryption
- **Monitoring**: Implement health checks and logging
- **Content Updates**: Expand `index.php` for full website functionality