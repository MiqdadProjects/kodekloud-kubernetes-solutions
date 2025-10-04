# 🌟 Task 2 - Deploy MySQL on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team needs to deploy a MySQL server on a Kubernetes cluster to support their application. After finalizing the requirements, the team has outlined the necessary configurations for a MySQL deployment, including persistent storage, secrets for credentials, and external access via a NodePort service. Your task is to create the required Kubernetes resources and ensure the deployment is operational.

### Requirements:
1. **PersistentVolume**: Create `mysql-pv` with a capacity of 250Mi.
2. **PersistentVolumeClaim**: Create `mysql-pv-claim` requesting 250Mi of storage.
3. **Deployment**: Create `mysql-deployment` using a MySQL image, with a volume mount at `/var/lib/mysql`.
4. **Service**: Create a NodePort service named `mysql` with `nodePort: 30007`.
5. **Secrets**:
   - `mysql-root-pass`: Key `password` with value `YUIidhb667`.
   - `mysql-user-pass`: Keys `username` with value `kodekloud_cap` and `password` with value `8FmzjvFU6S`.
   - `mysql-db-url`: Key `database` with value `kodekloud_db9`.
6. **Environment Variables**:
   - `MYSQL_ROOT_PASSWORD`: From `mysql-root-pass` secret, key `password`.
   - `MYSQL_DATABASE`: From `mysql-db-url` secret, key `database`.
   - `MYSQL_USER`: From `mysql-user-pass` secret, key `username`.
   - `MYSQL_PASSWORD`: From `mysql-user-pass` secret, key `password`.
7. **Verification**: Ensure the deployment and service are running and accessible.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Create the PersistentVolume, PersistentVolumeClaim, Secrets, Deployment, and Service configurations, apply them, and verify that the MySQL deployment is operational.

---

## 🔧 Infrastructure Overview

**Target Environment**: Kubernetes Cluster  
**Resources**:
- PersistentVolume: `mysql-pv` for storage
- PersistentVolumeClaim: `mysql-pv-claim` to request storage
- Secrets: `mysql-root-pass`, `mysql-user-pass`, `mysql-db-url` for credentials and database configuration
- Deployment: `mysql-deployment` for the MySQL application
- Service: `mysql` to expose the application via NodePort
- Namespace: Default

**Working Directory**: Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **PersistentVolume**: Provides 250Mi storage for MySQL data
- **PersistentVolumeClaim**: Requests storage from the PersistentVolume
- **Secrets**: Securely store MySQL credentials and database name
- **Deployment**: Runs the MySQL application with persistent storage and environment variables
- **Service**: Exposes MySQL on NodePort 30007
- **Application**: MySQL database accessible via port 3306

### 🎯 Implementation Strategy
1. Create a PersistentVolume with 250Mi capacity.
2. Create a PersistentVolumeClaim to request the PersistentVolume.
3. Create three Secrets to store MySQL credentials and database configuration.
4. Create a deployment with the MySQL image, volume mount, and environment variables.
5. Create a NodePort service to expose MySQL externally.
6. Apply the configurations and verify resource creation and application accessibility.

---

## 🚨 Potential Errors to Avoid

1. **PersistentVolume Misconfiguration**:
   **Issue**: Incorrect capacity or access mode in `mysql-pv`.
   **Fix**: Ensure capacity is `250Mi` and access mode is `ReadWriteOnce`.

2. **Secret Key Mismatch**:
   **Issue**: Incorrect key names or values in Secrets.
   **Fix**: Verify keys (`password`, `username`, `database`) and values match requirements.

3. **Environment Variable Misconfiguration**:
   **Issue**: Incorrect `secretKeyRef` names or keys in the deployment.
   **Fix**: Ensure environment variables reference the correct Secret names and keys.

4. **Volume Mount Issue**:
   **Issue**: Incorrect mount path or volume name.
   **Fix**: Confirm mount path is `/var/lib/mysql` and volume references `mysql-pv-claim`.

5. **Service Port Misconfiguration**:
   **Issue**: Incorrect `nodePort`, `port`, or `targetPort`.
   **Fix**: Set `nodePort: 30007`, `port: 3306`, and `targetPort: 3306`.

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Configuration File

#### MySQL Configuration (`mysql-deployment-task2.yaml`)
Create or edit a file named `mysql-deployment-task2.yaml`:

```yaml
# 1. PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  namespace: default
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql

---
# 2. PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi

---
# 3. Secrets
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
  namespace: default
type: Opaque
stringData:
  password: YUIidhb667

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
  namespace: default
type: Opaque
stringData:
  username: kodekloud_cap
  password: 8FmzjvFU6S

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
  namespace: default
type: Opaque
stringData:
  database: kodekloud_db9

---
# 4. Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
          name: mysql-port
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
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim

---
# 5. Service
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: default
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      nodePort: 30007
```

**Configuration Breakdown**:
- **PersistentVolume**:
  - **Name**: `mysql-pv`
  - **Namespace**: `default`
  - **Capacity**: `250Mi`
  - **Access Mode**: `ReadWriteOnce`
  - **Storage**: `hostPath` at `/mnt/data/mysql`
- **PersistentVolumeClaim**:
  - **Name**: `mysql-pv-claim`
  - **Namespace**: `default`
  - **Request**: `250Mi` with `ReadWriteOnce`
- **Secrets**:
  - `mysql-root-pass`: Key `password` with value `YUIidhb667`
  - `mysql-user-pass`: Keys `username` (`kodekloud_cap`) and `password` (`8FmzjvFU6S`)
  - `mysql-db-url`: Key `database` with value `kodekloud_db9`
- **Deployment**:
  - **Name**: `mysql-deployment`
  - **Namespace**: `default`
  - **Labels**: `app: mysql`
  - **Replicas**: 1
  - **Selector**: `matchLabels` with `app: mysql`
  - **Container**: `mysql` with image `mysql:5.7`
  - **Ports**: Exposes port 3306 (named `mysql-port`)
  - **Environment Variables**: References Secrets for `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD`
  - **Volume**: Mounts `mysql-pv-claim` at `/var/lib/mysql`
- **Service**:
  - **Name**: `mysql`
  - **Namespace**: `default`
  - **Type**: `NodePort`
  - **Selector**: `app: mysql`
  - **Ports**: `port: 3306`, `targetPort: 3306`, `nodePort: 30007`

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configurations
```bash
kubectl apply -f mysql-deployment-task2.yaml
```

**Purpose**: Create the PersistentVolume, PersistentVolumeClaim, Secrets, Deployment, and Service in the Kubernetes cluster.

**Expected Output**:
```
persistentvolume/mysql-pv created
persistentvolumeclaim/mysql-pv-claim created
secret/mysql-root-pass created
secret/mysql-user-pass created
secret/mysql-db-url created
deployment.apps/mysql-deployment created
service/mysql created
```

---

### Step 4: Verify Resources
```bash
kubectl get pv mysql-pv
```

**Expected Output**:
```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   2m
```

```bash
kubectl get pvc mysql-pv-claim
```

**Expected Output**:
```
NAME              STATUS   VOLUME     CAPACITY   ACCESS MODES   AGE
mysql-pv-claim    Bound    mysql-pv   250Mi      RWO            2m
```

```bash
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url
```

**Expected Output**:
```
NAME              TYPE     DATA   AGE
mysql-root-pass   Opaque   1      2m
mysql-user-pass   Opaque   2      2m
mysql-db-url      Opaque   1      2m
```

```bash
kubectl get deployments.apps mysql-deployment
```

**Expected Output**:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   1/1     1            1           2m
```

```bash
kubectl get pod -l app=mysql
```

**Expected Output**:
```
NAME                               READY   STATUS    RESTARTS   AGE
mysql-deployment-5d4b7c8f9-xyz123  1/1     Running   0          2m
```

```bash
kubectl get svc mysql
```

**Expected Output**:
```
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql   NodePort   10.96.123.456   <none>        3306:30007/TCP   2m
```

```bash
kubectl describe svc mysql
```

**Expected Output**:
```
Name:                     mysql
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=mysql
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.456
IPs:                      10.96.123.456
Port:                     <unset>  3306/TCP
TargetPort:               3306/TCP
NodePort:                 <unset>  30007/TCP
Endpoints:                10.244.0.7:3306
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

---

### Step 5: Verify MySQL Accessibility
```bash
POD=$(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- mysql -u kodekloud_cap -p8FmzjvFU6S -e "SELECT DATABASE();"
```

**Expected Output**:
```
+--------------------+
| DATABASE()         |
+--------------------+
| kodekloud_db9      |
+--------------------+
```

**Purpose**: Confirm the MySQL database is running and accessible with the provided credentials.

---

### Step 6: Verify PersistentVolume Mount
```bash
kubectl exec $POD -- ls /var/lib/mysql
```

**Expected Output**:
```
auto.cnf    ib_buffer_pool  ibdata1  ib_logfile0  ib_logfile1  kodekloud_db9
```

**Purpose**: Confirm the PersistentVolume is mounted at `/var/lib/mysql` and contains MySQL data.

---

## 🔍 Code Analysis

### MySQL Configuration (`mysql-deployment-task2.yaml`)
```yaml
# 1. PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  namespace: default
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql

---
# 2. PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi

---
# 3. Secrets
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
  namespace: default
type: Opaque
stringData:
  password: YUIidhb667

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
  namespace: default
type: Opaque
stringData:
  username: kodekloud_cap
  password: 8FmzjvFU6S

---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
  namespace: default
type: Opaque
stringData:
  database: kodekloud_db9

---
# 4. Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
          name: mysql-port
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
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim

---
# 5. Service
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: default
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      nodePort: 30007
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **PersistentVolume** | mysql-pv | capacity: 250Mi | Provides storage for MySQL data |
| **PersistentVolumeClaim** | mysql-pv-claim | request: 250Mi | Requests storage from PersistentVolume |
| **Secret (mysql-root-pass)** | password | YUIidhb667 | Root password for MySQL |
| **Secret (mysql-user-pass)** | username, password | kodekloud_cap, 8FmzjvFU6S | User credentials for MySQL |
| **Secret (mysql-db-url)** | database | kodekloud_db9 | Database name for MySQL |
| **Deployment** | mysql-deployment | replicas: 1, app: mysql | Runs MySQL application |
| **Container** | mysql | image: mysql:5.7 | MySQL container |
| **Container Port** | mysql-port | 3306 | Default MySQL port |
| **Volume** | mysql-storage | PVC: mysql-pv-claim | Mounted at /var/lib/mysql |
| **Service** | mysql | type: NodePort | Exposes MySQL externally |
| **Service Ports** | port, targetPort, nodePort | 3306, 3306, 30007 | Routes traffic to MySQL |

---

## ✅ Verification Steps

### Step 1: Verify PersistentVolume
```bash
kubectl get pv mysql-pv
```

**Expected Output**:
```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   2m
```

### Step 2: Verify PersistentVolumeClaim
```bash
kubectl get pvc mysql-pv-claim
```

**Expected Output**:
```
NAME              STATUS   VOLUME     CAPACITY   ACCESS MODES   AGE
mysql-pv-claim    Bound    mysql-pv   250Mi      RWO            2m
```

### Step 3: Verify Secrets
```bash
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url
```

**Expected Output**:
```
NAME              TYPE     DATA   AGE
mysql-root-pass   Opaque   1      2m
mysql-user-pass   Opaque   2      2m
mysql-db-url      Opaque   1      2m
```

### Step 4: Verify Deployment
```bash
kubectl get deployments.apps mysql-deployment
```

**Expected Output**:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   1/1     1            1           2m
```

### Step 5: Verify Pods
```bash
kubectl get pod -l app=mysql
```

**Expected Output**:
```
NAME                               READY   STATUS    RESTARTS   AGE
mysql-deployment-5d4b7c8f9-xyz123  1/1     Running   0          2m
```

### Step 6: Verify Service
```bash
kubectl get svc mysql
```

**Expected Output**:
```
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql   NodePort   10.96.123.456   <none>        3306:30007/TCP   2m
```

### Step 7: Verify Service Endpoints
```bash
kubectl describe svc mysql
```

**Expected Output**:
```
Name:                     mysql
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=mysql
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.456
IPs:                      10.96.123.456
Port:                     <unset>  3306/TCP
TargetPort:               3306/TCP
NodePort:                 <unset>  30007/TCP
Endpoints:                10.244.0.7:3306
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Step 8: Verify MySQL Database
```bash
POD=$(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- mysql -u kodekloud_cap -p8FmzjvFU6S -e "SELECT DATABASE();"
```

**Expected Output**:
```
+--------------------+
| DATABASE()         |
+--------------------+
| kodekloud_db9      |
+--------------------+
```

---

## 🧪 Testing

### Verify MySQL Port
```bash
POD=$(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN
```

**Purpose**: Confirm MySQL is listening on port 3306.

### Verify PersistentVolume Mount
```bash
kubectl exec $POD -- ls /var/lib/mysql
```

**Expected Output**:
```
auto.cnf    ib_buffer_pool  ibdata1  ib_logfile0  ib_logfile1  kodekloud_db9
```

**Purpose**: Confirm the PersistentVolume is correctly mounted.

### Verify MySQL Connectivity
```bash
kubectl exec -it $POD -- mysql -u kodekloud_cap -p8FmzjvFU6S -e "SHOW DATABASES;"
```

**Expected Output**:
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kodekloud_db9      |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

**Purpose**: Confirm the database `kodekloud_db9` is created and accessible.

---

## 📚 Quick Command Reference
```bash
# Apply configurations
kubectl apply -f mysql-deployment-task2.yaml

# Verify resources
kubectl get pv mysql-pv
kubectl get pvc mysql-pv-claim
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url
kubectl get deployments.apps mysql-deployment
kubectl get pod -l app=mysql
kubectl get svc mysql

# Verify MySQL database
POD=$(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- mysql -u kodekloud_cap -p8FmzjvFU6S -e "SELECT DATABASE();"

# Verify PersistentVolume mount
kubectl exec $POD -- ls /var/lib/mysql

# Test MySQL connectivity
kubectl exec -it $POD -- mysql -u kodekloud_cap -p8FmzjvFU6S -e "SHOW DATABASES;"
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.  
**Solution**: Check pod events and logs.
```bash
kubectl describe pod -l app=mysql
kubectl logs -l app=mysql
```

### Issue 2: PersistentVolumeClaim Not Bound
**Symptoms**: `kubectl get pvc` shows `Pending`.  
**Solution**: Verify PersistentVolume configuration and availability.
```bash
kubectl describe pv mysql-pv
kubectl describe pvc mysql-pv-claim
```

### Issue 3: Service Has No Endpoints
**Symptoms**: `kubectl describe svc mysql` shows `Endpoints: <none>`.  
**Solution**: Verify selector and port alignment.
```bash
kubectl get pod -l app=mysql -o wide
kubectl describe svc mysql
```

### Issue 4: Image Pull Failure
**Symptoms**: Pod fails with `ImagePullBackOff`.  
**Solution**: Verify the image name (`mysql:5.7`).
```bash
kubectl describe pod -l app=mysql
```

### Issue 5: MySQL Connection Failure
**Symptoms**: `mysql` command fails with authentication errors.  
**Solution**: Verify Secret values and environment variables.
```bash
kubectl describe secret mysql-root-pass mysql-user-pass mysql-db-url
kubectl describe deployment mysql-deployment
```

---

## 💡 Additional Tips
- **Pod Logs**: Check for MySQL errors.
  ```bash
  kubectl logs -l app=mysql
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f mysql-deployment-task2.yaml --dry-run=client
  ```
- **NodePort Range**: Ensure 30007 is within 30000-32767.
- **MySQL Client**: Test database operations.
  ```bash
  POD=$(kubectl get pod -l app=mysql -o jsonpath='{.items[0].metadata.name}')
  kubectl exec -it $POD -- mysql -u kodekloud_cap -p8FmzjvFU6S
  ```
- **Volume Verification**: Confirm data persistence.
  ```bash
  kubectl exec $POD -- ls /var/lib/mysql
  ```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Persistent Storage**: Configuring the PersistentVolume and PersistentVolumeClaim correctly.
2. **Secrets Management**: Ensuring all Secrets have the correct key-value pairs.
3. **Environment Variables**: Properly linking Secrets to environment variables.
4. **Service Exposure**: Setting up the NodePort service with the correct port.

### 💡 Solution Approach:
1. **Persistent Storage**: Created `mysql-pv` and `mysql-pv-claim` with 250Mi capacity and `ReadWriteOnce` access mode.
2. **Secrets**: Defined three Secrets with the exact key-value pairs specified.
3. **Deployment**: Configured `mysql-deployment` with `mysql:5.7`, environment variables from Secrets, and volume mount at `/var/lib/mysql`.
4. **Service**: Set up `mysql` service as NodePort with `nodePort: 30007`.
5. **Verification**: Confirmed resource creation, database accessibility, and volume mount.

### 🎯 Key Success Factors:
- **PersistentVolume**: Provides reliable storage for MySQL data.
- **Secrets**: Securely manage credentials and database configuration.
- **Deployment**: Correctly configured with environment variables and volume.
- **Service**: Exposes MySQL on the specified NodePort.
- **Application**: Database `kodekloud_db9` is created and accessible.

### ⚠️ Critical Configuration Details:
- **PersistentVolume**: `mysql-pv`, 250Mi
- **PersistentVolumeClaim**: `mysql-pv-claim`, 250Mi
- **Secrets**: `mysql-root-pass`, `mysql-user-pass`, `mysql-db-url`
- **Deployment**: `mysql-deployment`, image `mysql:5.7`
- **Service**: `mysql`, NodePort 30007
- **Mount Path**: `/var/lib/mysql`

### 🔒 Kubernetes Benefits:
- **Persistence**: PersistentVolume ensures data durability.
- **Security**: Secrets securely manage sensitive data.
- **Accessibility**: NodePort enables external access.
- **Reliability**: Kubernetes manages pod health and restarts.

---

## ⚠️ Important Production Notes
🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.  
🔐 **Security**: Implement network policies to restrict MySQL access.  
📊 **Monitoring**: Monitor MySQL performance and disk usage.  
🛡️ **Backup**: Configure backups for the PersistentVolume.  
🌐 **Production Storage**: Replace `hostPath` with a production-grade storage class.

---

## 📖 Learning Outcomes
**Key Concepts Mastered**:
- **PersistentVolume and PersistentVolumeClaim**: Managing storage in Kubernetes.
- **Secrets**: Securely storing and referencing sensitive data.
- **Deployment**: Configuring MySQL with environment variables and volumes.
- **Service**: Exposing applications via NodePort.
- **Troubleshooting**: Verifying resource configurations and application health.

**Kubernetes Features Used**:
- `v1 PersistentVolume` and `v1 PersistentVolumeClaim` for storage.
- `v1 Secret` for credential management.
- `apps/v1 Deployment` for MySQL application.
- `v1 Service` for NodePort exposure.

---

## 🎯 Task Completion Summary
✅ **Successfully Completed**:
- **PersistentVolume** - Created `mysql-pv` with 250Mi capacity.
- **PersistentVolumeClaim** - Created `mysql-pv-claim` requesting 250Mi.
- **Secrets** - Created `mysql-root-pass`, `mysql-user-pass`, and `mysql-db-url` with specified values.
- **Deployment** - Created `mysql-deployment` with `mysql:5.7`, volume mount, and environment variables.
- **Service** - Created `mysql` service with NodePort 30007.
- **Verification** - Confirmed resources are running and MySQL is accessible with the correct database.

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Application Foundation**: The MySQL deployment is operational, with persistent storage, secure credentials, and external access via NodePort 30007, ready for testing and future production enhancements by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Testing**: Validate database performance and connectivity.
- **Scaling**: Consider read replicas for high availability.
- **Security**: Add network policies and encryption.
- **Monitoring**: Implement health checks and metrics collection.
- **Backups**: Set up regular database backups.