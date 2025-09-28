# 🌟 Task 7 - Deploy LEMP Stack Website on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team aims to deploy a static website on a Kubernetes cluster using Nginx, PHP-FPM, and MySQL. The setup requires secure storage of MySQL credentials, configuration for PHP, and proper exposure of the web application. The task includes creating secrets, a ConfigMap, a deployment, services, and copying a PHP file to the application’s document root.

### Requirements:
- Create four secrets for MySQL:
  - **`mysql-root-pass`** with key `password` and value `R00t`.
  - **`mysql-user-pass`** with keys `username` (value `kodekloud_gem`) and `password` (value `dCV3szSGNA`).
  - **`mysql-db-url`** with key `database` and value `kodekloud_db4`.
  - **`mysql-host`** with key `host` and value `mysql-service`.
- Create a ConfigMap named **`php-config`** for `php.ini` with `variables_order = "EGPCS"`.
- Create a deployment named **`lemp-wp`** with:
  - 1 replica and label `app: lemp-wp`.
  - Two containers:
    - **`nginx-php-container`** using `webdevops/php-nginx:alpine-3-php7`, with `php-config` mounted at `/opt/docker/etc/php/php.ini`.
    - **`mysql-container`** using `mysql:5.6`.
  - Environment variables for both containers: `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_HOST`, sourced from the secrets.
- Create a NodePort service named **`lemp-service`** on port `30008` to expose the web application.
- Create a ClusterIP service named **`mysql-service`** on port `3306` for MySQL.
- Copy `/tmp/index.php` from the jump host to the `nginx-php-container` under `/app`, replacing dummy MySQL values with environment variables.
- Ensure the website is accessible via the **Website** button and displays **"Connected successfully"**.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. The `/tmp/index.php` file exists on the jump host.

👉 **Your task**: Deploy a LEMP stack website on the Kubernetes cluster, ensuring secure MySQL configuration and application accessibility.

Note: The `/tmp/index.php` file exists on the jump host and according to task we need to mofify it , and if permission issues arise, use `sudo chmod 777 /tmp/index.php` with the password `mjolnir123`.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- Secrets for MySQL credentials
- ConfigMap for PHP configuration
- Deployment with Nginx/PHP-FPM and MySQL containers
- NodePort and ClusterIP services
- PHP application file (`index.php`)

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Secrets**: Securely store MySQL credentials
- **ConfigMap**: Stores PHP configuration
- **Deployment**: Runs Nginx/PHP-FPM and MySQL containers
- **Services**: Expose web application and MySQL
- **Application**: PHP website connects to MySQL using environment variables

### 🎯 Implementation Strategy
1. Create secrets for MySQL credentials
2. Create a ConfigMap for PHP configuration
3. Deploy a LEMP stack with two containers
4. Configure services for web and MySQL access
5. Copy and configure `index.php` with environment variables
6. Verify deployment and application accessibility

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Configuration File
Create a file named `lemp-deployment.yaml`:

```yaml
# lemp-deployment.yaml
# Define secrets for MySQL credentials
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: R00t
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: kodekloud_gem
  password: dCV3szSGNA
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: kodekloud_db4
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-host
type: Opaque
stringData:
  host: mysql-service
---
# Define ConfigMap for PHP configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
data:
  php.ini: |
    variables_order = "EGPCS"
---
# Define deployment for LEMP stack
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lemp-wp
  labels:
    app: lemp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lemp-wp
  template:
    metadata:
      labels:
        app: lemp-wp
    spec:
      containers:
        - name: nginx-php-container
          image: webdevops/php-nginx:alpine-3-php7
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - mountPath: /opt/docker/etc/php/php.ini
              name: php-ini
              subPath: php.ini
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-host
                  key: host
        - name: mysql-container
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-host
                  key: host
      volumes:
        - name: php-ini
          configMap:
            name: php-config
---
# Define NodePort service for web application
apiVersion: v1
kind: Service
metadata:
  name: lemp-service
spec:
  selector:
    app: lemp-wp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
  type: NodePort
---
# Define ClusterIP service for MySQL
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: lemp-wp
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

**Configuration Breakdown:**
- **Secrets**:
  - `mysql-root-pass`: Stores root password (`R00t`)
  - `mysql-user-pass`: Stores username (`kodekloud_gem`) and password (`dCV3szSGNA`)
  - `mysql-db-url`: Stores database name (`kodekloud_db4`)
  - `mysql-host`: Stores host (`mysql-service`)
- **ConfigMap**:
  - `name: php-config` - Defines `php.ini` with `variables_order = "EGPCS"`
- **Deployment**:
  - `name: lemp-wp` - Deployment name with 1 replica
  - `labels: app: lemp-wp` - Matches selector for pod management
  - `nginx-php-container`: Uses `webdevops/php-nginx:alpine-3-php7`, mounts ConfigMap
  - `mysql-container`: Uses `mysql:5.6`
  - Environment variables sourced from secrets for both containers
- **Services**:
  - `lemp-service`: NodePort on 30008 for web access
  - `mysql-service`: ClusterIP on 3306 for MySQL

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configuration
```bash
kubectl apply -f lemp-deployment.yaml
```

**Purpose:** Deploy secrets, ConfigMap, deployment, and services.

**Expected Output:**
```
secret/mysql-root-pass created
secret/mysql-user-pass created
secret/mysql-db-url created
secret/mysql-host created
configmap/php-config created
deployment.apps/lemp-wp created
service/lemp-service created
service/mysql-service created
```

---

### Step 4: Modify and Copy `index.php`
Modify `/tmp/index.php` to use environment variables:

```bash
cd /tmp
sudo chmod 777 /tmp/index.php   **Password:** `mjolnir123
vi index.php
```


**Content of `index.php`**:
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

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

Copy `index.php` to the container:
```bash
POD=$(kubectl get pod -l app=lemp-wp -o jsonpath='{.items[0].metadata.name}')
kubectl cp /tmp/index.php $POD:/app -c nginx-php-container
```

**Purpose:** Copy the PHP file to the Nginx document root (`/app`) and ensure it uses environment variables.

---

### Step 5: Verify Resources
```bash
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url mysql-host
```

**Expected Output:**
```
NAME              TYPE     DATA   AGE
mysql-root-pass   Opaque   1      2m
mysql-user-pass   Opaque   2      2m
mysql-db-url      Opaque   1      2m
mysql-host        Opaque   1      2m
```

```bash
kubectl get configmap php-config
```

**Expected Output:**
```
NAME         DATA   AGE
php-config   1      2m
```

```bash
kubectl get deploy lemp-wp
```

**Expected Output:**
```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
lemp-wp   1/1     1            1           2m
```

```bash
kubectl get pod -l app=lemp-wp
```

**Expected Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE
lemp-wp-6df8f7969f-lch4c  2/2     Running   0          2m
```

```bash
kubectl get svc lemp-service mysql-service
```

**Expected Output:**
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
lemp-service    NodePort    10.96.123.456   <none>        80:30008/TCP     2m
mysql-service   ClusterIP   10.96.123.457   <none>        3306/TCP          2m
```

---

### Step 6: Verify Application Accessibility
**Access the Application**:
- Click the **Website** button in the lab interface.
- Alternatively, use the cluster’s node IP:
```bash
curl http://<node-ip>:30008
```

**Expected Output:**
```
Connected successfully
```

**Verification:** Confirm the website displays "Connected successfully".

---

## 🔍 Code Analysis

### Complete Configuration (`lemp-deployment.yaml`)
```yaml
# Define secrets for MySQL credentials
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: R00t
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: kodekloud_gem
  password: dCV3szSGNA
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: kodekloud_db4
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-host
type: Opaque
stringData:
  host: mysql-service
---
# Define ConfigMap for PHP configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-config
data:
  php.ini: |
    variables_order = "EGPCS"
---
# Define deployment for LEMP stack
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lemp-wp
  labels:
    app: lemp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lemp-wp
  template:
    metadata:
      labels:
        app: lemp-wp
    spec:
      containers:
        - name: nginx-php-container
          image: webdevops/php-nginx:alpine-3-php7
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - mountPath: /opt/docker/etc/php/php.ini
              name: php-ini
              subPath: php.ini
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-host
                  key: host
        - name: mysql-container
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-host
                  key: host
      volumes:
        - name: php-ini
          configMap:
            name: php-config
---
# Define NodePort service for web application
apiVersion: v1
kind: Service
metadata:
  name: lemp-service
spec:
  selector:
    app: lemp-wp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
  type: NodePort
---
# Define ClusterIP service for MySQL
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: lemp-wp
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
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
| **Secret** | mysql-root-pass | password: R00t | Root password |
| **Secret** | mysql-user-pass | username: kodekloud_gem, password: dCV3szSGNA | User credentials |
| **Secret** | mysql-db-url | database: kodekloud_db4 | Database name |
| **Secret** | mysql-host | host: mysql-service | MySQL service host |
| **ConfigMap** | php-config | variables_order = "EGPCS" | PHP configuration |
| **Deployment** | lemp-wp | replicas: 1, app: lemp-wp | Runs LEMP stack |
| **nginx-php-container** | image | webdevops/php-nginx:alpine-3-php7 | Nginx with PHP-FPM |
| **mysql-container** | image | mysql:5.6 | MySQL database |
| **lemp-service** | type | NodePort | Exposes web on 30008 |
| **mysql-service** | type | ClusterIP | Exposes MySQL on 3306 |

---

## ✅ Verification Steps

### Step 1: Verify Secrets
```bash
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url mysql-host
```

**Expected Output:**
```
NAME              TYPE     DATA   AGE
mysql-root-pass   Opaque   1      2m
mysql-user-pass   Opaque   2      2m
mysql-db-url      Opaque   1      2m
mysql-host        Opaque   1      2m
```

### Step 2: Verify ConfigMap
```bash
kubectl get configmap php-config
```

**Expected Output:**
```
NAME         DATA   AGE
php-config   1      2m
```

### Step 3: Verify Deployment
```bash
kubectl get deploy lemp-wp
```

**Expected Output:**
```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
lemp-wp   1/1     1            1           2m
```

### Step 4: Verify Pod
```bash
kubectl get pod -l app=lemp-wp
```

**Expected Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE
lemp-wp-6df8f7969f-lch4c  2/2     Running   0          2m
```

### Step 5: Verify Services
```bash
kubectl get svc lemp-service mysql-service
```

**Expected Output:**
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
lemp-service    NodePort    10.96.123.456   <none>        80:30008/TCP     2m
mysql-service   ClusterIP   10.96.123.457   <none>        3306/TCP          2m
```

### Step 6: Verify Application
```bash
CLICK ON WEBSITE BUTTON

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
POD=$(kubectl get pod -l app=lemp-wp -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -c nginx-php-container -- cat /opt/docker/etc/php/php.ini
```

**Expected Output:**
```
variables_order = "EGPCS"
```

### Verify Environment Variables
```bash
kubectl exec $POD -c nginx-php-container -- env | grep MYSQL
```

**Expected Output:**
```
MYSQL_ROOT_PASSWORD=R00t
MYSQL_DATABASE=kodekloud_db4
MYSQL_USER=kodekloud_gem
MYSQL_PASSWORD=dCV3szSGNA
MYSQL_HOST=mysql-service
```

### Verify index.php
```bash
kubectl exec $POD -c nginx-php-container -- cat /app/index.php
```

**Purpose:** Confirm `index.php` uses environment variables.

### Verify MySQL Service
```bash
kubectl describe svc mysql-service
```

**Purpose:** Confirm MySQL service configuration.

---

## 📚 Quick Command Reference

```bash
# Apply configuration
kubectl apply -f lemp-deployment.yaml

# Copy index.php
cd /tmp
POD=$(kubectl get pod -l app=lemp-wp -o jsonpath='{.items[0].metadata.name}')
kubectl cp index.php $POD:/app -c nginx-php-container

# Verify resources
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url mysql-host
kubectl get configmap php-config
kubectl get deploy lemp-wp
kubectl get pod -l app=lemp-wp
kubectl get svc lemp-service mysql-service

# Test application
curl http://<node-ip>:30008
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms:** Pods in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod -l app=lemp-wp
kubectl logs -l app=lemp-wp -c nginx-php-container
kubectl logs -l app=lemp-wp -c mysql-container
```

### Issue 2: Database Connectivity Failure
**Symptoms:** Application shows "Connection failed".
**Solution:** Verify environment variables and MySQL service.
```bash
kubectl exec $POD -c nginx-php-container -- env | grep MYSQL
kubectl describe svc mysql-service
```

### Issue 3: Incorrect NodePort
**Symptoms:** `lemp-service` uses wrong port.
**Solution:** Patch the service.
```bash
kubectl patch svc lemp-service -p '{"spec":{"ports":[{"port":80,"targetPort":80,"nodePort":30008}]}}'
```

### Issue 4: ConfigMap Not Mounted
**Symptoms:** PHP configuration not applied.
**Solution:** Verify mount path.
```bash
kubectl exec $POD -c nginx-php-container -- cat /opt/docker/etc/php/php.ini
```

### Issue 5: File Copy Failure
**Symptoms:** `kubectl cp` fails or `index.php` missing.
**Solution:** Verify pod name and permissions.
```bash
kubectl get pod -l app=lemp-wp
kubectl exec $POD -c nginx-php-container -- ls /app
```

---

## 💡 Additional Tips

- **Pod Logs**: Check for runtime issues.
  ```bash
  kubectl logs -l app=lemp-wp -c nginx-php-container
  kubectl logs -l app=lemp-wp -c mysql-container
  ```
- **Dry Run**: Test configuration before applying.
  ```bash
  kubectl apply -f lemp-deployment.yaml --dry-run=client
  ```
- **MySQL Health**: Verify MySQL container status.
  ```bash
  kubectl exec $POD -c mysql-container -- mysqladmin -u kodekloud_gem -pdCV3szSGNA status
  ```
- **NodePort Range**: Ensure 30008 is within 30000-32767.

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Secrets Management**: Creating multiple secrets with specific key/value pairs.
2. **ConfigMap Mounting**: Correctly mounting `php.ini` in the Nginx container.
3. **Environment Variables**: Sourcing variables from secrets.
4. **File Configuration**: Replacing dummy values in `index.php` with environment variables.
5. **Service Exposure**: Ensuring correct NodePort and ClusterIP settings.

### 💡 Solution Approach:
1. **Secrets**: Created four secrets with specified values.
2. **ConfigMap**: Configured `php-config` for `php.ini`.
3. **Deployment**: Set up `lemp-wp` with Nginx and MySQL containers.
4. **Services**: Exposed web app on NodePort 30008 and MySQL on ClusterIP 3306.
5. **Application**: Modified and copied `index.php` to use environment variables.
6. **Verification**: Confirmed resources and application output.

### 🎯 Key Success Factors:
- **Secrets**: Securely store MySQL credentials
- **ConfigMap**: Mounted correctly at `/opt/docker/etc/php/php.ini`
- **Deployment**: Runs two containers with environment variables
- **Services**: Correctly expose application and database
- **Application**: Displays "Connected successfully"

### ⚠️ Critical Configuration Details:
- **Secrets**: `mysql-root-pass`, `mysql-user-pass`, `mysql-db-url`, `mysql-host`
- **ConfigMap**: `php-config`
- **Deployment**: `lemp-wp`
- **Services**: `lemp-service` (NodePort 30008), `mysql-service` (ClusterIP 3306)
- **Application Path**: `/app/index.php`

### 🔒 Kubernetes Benefits:
- **Security**: Secrets protect sensitive data
- **Scalability**: Deployment supports replicas
- **Accessibility**: NodePort enables external access
- **Reliability**: Kubernetes manages pod health

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.

🔐 **Security**: Implement network policies for MySQL access.

📊 **Monitoring**: Monitor Nginx and MySQL health.

🛡️ **Scalability**: Use separate MySQL deployment for production.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **Secrets**: Secure storage of credentials
- **ConfigMap**: Managing configuration files
- **Multi-Container Pods**: Running Nginx/PHP-FPM and MySQL
- **Services**: Exposing applications and databases
- **Environment Variables**: Sourcing from secrets
- **File Management**: Copying files with `kubectl cp`

**Kubernetes Features Used:**
- `v1 Secret` for credentials
- `v1 ConfigMap` for PHP configuration
- `apps/v1 Deployment` for multi-container pods
- `v1 Service` for NodePort and ClusterIP
- `kubectl cp` for file transfer

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **Secrets** - Created `mysql-root-pass`, `mysql-user-pass`, `mysql-db-url`, `mysql-host`
- **ConfigMap** - Created `php-config` for `php.ini`
- **Deployment** - Created `lemp-wp` with Nginx and MySQL
- **Services** - Created `lemp-service` (NodePort 30008) and `mysql-service` (ClusterIP 3306)
- **Application** - Copied `index.php` with environment variables
- **Verification** - Confirmed "Connected successfully"

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The LEMP stack website is deployed, accessible via NodePort 30008, and displays "Connected successfully", ready for further enhancements by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Scaling**: Increase replicas or use autoscaling
- **Security**: Add network policies and secret encryption
- **Monitoring**: Implement health checks and logging
- **Content Updates**: Expand `index.php` for full website functionality
