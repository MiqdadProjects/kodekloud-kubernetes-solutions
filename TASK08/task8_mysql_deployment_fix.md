# 🌟 Task 8 - Fix MySQL Deployment Template on Kubernetes Cluster

## 📌 Task Description

One of the Nautilus DevOps team members was working on updating an existing Kubernetes template (`mysql_deployment.yml`). Due to errors in the template, it is failing to apply. The task is to fix the template without removing any components (e.g., PersistentVolume, PersistentVolumeClaim, Deployment, Service, volumes) and ensure it can be applied successfully using `kubectl`. The template is located at `/home/thor/mysql_deployment.yml` on the jump host.

### Requirements:

- Fix the errors in `/home/thor/mysql_deployment.yml`.
- Ensure all components (PersistentVolume, PersistentVolumeClaim, Service, Deployment, volumes) are retained.
- Apply the corrected YAML and verify the deployment.
- The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

**Note**: Do not remove any components from the template.

👉 **Your task**: Fix the `mysql_deployment.yml` template, apply it, and verify that all resources are created and running correctly.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster **Resources:**

- PersistentVolume for MySQL data storage
- PersistentVolumeClaim to request storage
- Service to expose MySQL
- Deployment for the MySQL container
- Volumes for persistent storage

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components

- **PersistentVolume**: Provides storage for MySQL data
- **PersistentVolumeClaim**: Requests storage for the deployment
- **Service**: Exposes MySQL on NodePort 30011
- **Deployment**: Runs the MySQL container with environment variables from secrets
- **Volumes**: Mounts persistent storage for MySQL

### 🎯 Implementation Strategy

1. Identify and fix errors in the `mysql_deployment.yml` template
2. Apply the corrected YAML to the Kubernetes cluster
3. Verify the creation of PersistentVolume, PersistentVolumeClaim, Service, Deployment, and Pods
4. Ensure the MySQL deployment is running correctly

---

## 🚨 Errors in the Original YAML

The original `mysql_deployment.yml` contains the following errors, which need to be corrected:

 1. **Wrong apiVersion for PersistentVolume**:

    ```yaml
    apiVersion: apps/v1
    kind: PersistentVolume
    ```

    **Issue**: PersistentVolume is a core API resource and should use `apiVersion: v1`. **Fix**: Change to `apiVersion: v1`.

 2. **Misplaced labels in PersistentVolume**:

    ```yaml
    labels:
      type: local
    ```

    **Issue**: Indentation is correct, but it’s worth ensuring clarity to avoid parsing issues. **Fix**: Ensure proper indentation under `metadata`.

 3. **hostPath indentation error**:

    ```yaml
    hostPath:
      path: "/mnt/data"
    ```

    **Issue**: The `path` field was not indented correctly under `hostPath`. **Fix**: Properly indent `path` under `hostPath`.

 4. **Wrong format for persistentVolumeReclaimPolicy**:

    ```yaml
    persistentVolumeReclaimPolicy:
      - Retain
    ```

    **Issue**: This field takes a string, not a list. **Fix**: Use `persistentVolumeReclaimPolicy: Retain`.

 5. **PVC storage unit typo**:

    ```yaml
    storage: 250MB
    ```

    **Issue**: Kubernetes uses binary SI units (e.g., Mi, Gi), not MB. **Fix**: Change to `storage: 250Mi`.

 6. **Wrong apiVersion for Deployment**:

    ```yaml
    apiVersion: v1
    kind: Deployment
    ```

    **Issue**: Deployments are under `apps/v1`, not `v1`. **Fix**: Change to `apiVersion: apps/v1`.

 7. **Misplaced tier labels in selector**:

    ```yaml
    selector:
      matchLabels:
        app: mysql-app
      tier: mysql
    ```

    **Issue**: `tier` is incorrectly placed outside `matchLabels`. **Fix**: Move `tier: mysql` under `matchLabels`.

 8. **Wrong field name in Deployment container**:

    ```yaml
    - images: mysql:5.6
    ```

    **Issue**: The field should be `image`, not `images`. **Fix**: Change to `image: mysql:5.6`.

 9. **Indentation of secretKeyRef in env**:

    ```yaml
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
      secretKeyRef:
        name: mysql-root-pass
          key: password
    ```

    **Issue**: `secretKeyRef` was not properly indented under `valueFrom`, and `key` was indented too far. **Fix**: Correct indentation for `secretKeyRef` and its subfields.

10. **Volumes section mis-indented**:

    ```yaml
    volumes:
    - name: mysql-persistent-storage
        persistentVolumeClaim:
        claimName: mysql-pv-claim
    ```

    **Issue**: `persistentVolumeClaim` was not properly indented under the volume. **Fix**: Correct indentation for `persistentVolumeClaim`.

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands and edit the YAML file.

---

### Step 2: Create Corrected Configuration File

Edit the file `/home/thor/mysql_deployment.yml` with the corrected YAML:

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: mysql-app
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql-app
spec:
  type: NodePort
  ports:
    - targetPort: 3306
      port: 3306
      nodePort: 30011
  selector:
    app: mysql-app
    tier: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql-app
spec:
  selector:
    matchLabels:
      app: mysql-app
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql-app
        tier: mysql
    spec:
      containers:
        - name: mysql
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
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

**Configuration Breakdown**:

- **PersistentVolume**:
  - `name: mysql-pv` - Provides 250Mi storage at `/mnt/data`
  - `labels: type: local` - Identifies the storage type
  - `persistentVolumeReclaimPolicy: Retain` - Retains data after PVC deletion
- **PersistentVolumeClaim**:
  - `name: mysql-pv-claim` - Requests 250Mi storage
  - `labels: app: mysql-app` - Associates with the MySQL app
- **Service**:
  - `name: mysql` - NodePort service on port 30011 for MySQL access
  - `selector: app: mysql-app, tier: mysql` - Targets MySQL pods
- **Deployment**:
  - `name: mysql-deployment` - Runs MySQL container
  - `labels: app: mysql-app, tier: mysql` - Matches selector for pod management
  - `image: mysql:5.6` - MySQL container image
  - Environment variables sourced from secrets
  - Volume mounts for persistent storage at `/var/lib/mysql`

**Save and Exit**: In `vi`, press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configuration

```bash
kubectl apply -f /home/thor/mysql_deployment.yml
```

**Purpose**: Deploy the PersistentVolume, PersistentVolumeClaim, Service, and Deployment.

**Expected Output**:

```
persistentvolume/mysql-pv created
persistentvolumeclaim/mysql-pv-claim created
service/mysql created
deployment.apps/mysql-deployment created
```

---

### Step 4: Verify Resources

```bash
kubectl get pv,pvc
```

**Expected Output**:

```
NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
persistentvolume/mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   standard                2m

NAME                                STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            standard       2m
```

```bash
kubectl get svc mysql
```

**Expected Output**:

```
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql   NodePort   10.96.123.456   <none>        3306:30011/TCP   2m
```

```bash
kubectl get deploy mysql-deployment
```

**Expected Output**:

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   1/1     1            1           2m
```

```bash
kubectl get pod -l app=mysql-app
```

**Expected Output**:

```
NAME                               READY   STATUS    RESTARTS   AGE
mysql-deployment-5f8f7b6d9c-xyz   1/1     Running   0          2m
```

---

### Step 5: Verify MySQL Accessibility

**Test MySQL Service**:

```bash
POD=$(kubectl get pod -l app=mysql-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- mysqladmin -u kodekloud_gem -pdCV3szSGNA status
```

**Expected Output**:

```
Uptime: 120  Threads: 1  Questions: 1  Slow queries: 0  Opens: 1  Flush tables: 1  Open tables: 0  Queries per second avg: 0.008
```

**Purpose**: Confirm the MySQL container is running and accessible.

---

## 🔍 Code Analysis

### Complete Configuration (`mysql_deployment.yml`)

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: mysql-app
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql-app
spec:
  type: NodePort
  ports:
    - targetPort: 3306
      port: 3306
      nodePort: 30011
  selector:
    app: mysql-app
    tier: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql-app
spec:
  selector:
    matchLabels:
      app: mysql-app
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql-app
        tier: mysql
    spec:
      containers:
        - name: mysql
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
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

### Resource Attributes

| Resource | Attribute | Value | Description |
| --- | --- | --- | --- |
| **PersistentVolume** | mysql-pv | storage: 250Mi | Provides storage at `/mnt/data` |
| **PersistentVolumeClaim** | mysql-pv-claim | storage: 250Mi | Requests storage for MySQL |
| **Service** | mysql | type: NodePort | Exposes MySQL on port 30011 |
| **Deployment** | mysql-deployment | image: mysql:5.6 | Runs MySQL container |
| **mysql-container** | image | mysql:5.6 | MySQL database container |
| **Volume** | mysql-persistent-storage | claimName: mysql-pv-claim | Mounts PVC at `/var/lib/mysql` |

---

## ✅ Verification Steps

### Step 1: Verify PersistentVolume and PersistentVolumeClaim

```bash
kubectl get pv,pvc
```

**Expected Output**:

```
NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
persistentvolume/mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   standard                2m

NAME                                STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            standard       2m
```

### Step 2: Verify Service

```bash
kubectl get svc mysql
```

**Expected Output**:

```
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql   NodePort   10.96.123.456   <none>        3306:30011/TCP   2m
```

### Step 3: Verify Deployment

```bash
kubectl get deploy mysql-deployment
```

**Expected Output**:

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment   1/1     1            1           2m
```

### Step 4: Verify Pod

```bash
kubectl get pod -l app=mysql-app
```

**Expected Output**:

```
NAME                               READY   STATUS    RESTARTS   AGE
mysql-deployment-5f8f7b6d9c-xyz   1/1     Running   0          2m
```

### Step 5: Verify MySQL Health

```bash
POD=$(kubectl get pod -l app=mysql-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- mysqladmin -u kodekloud_gem -pdCV3szSGNA status
```

**Expected Output**:

```
Uptime: 120  Threads: 1  Questions: 1  Slow queries: 0  Opens: 1  Flush tables: 1  Open tables: 0  Queries per second avg: 0.008
```

---

## 🧪 Testing

### Verify Environment Variables

```bash
POD=$(kubectl get pod -l app=mysql-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- env | grep MYSQL
```

**Expected Output**:

```
MYSQL_ROOT_PASSWORD=<value>
MYSQL_DATABASE=<value>
MYSQL_USER=kodekloud_gem
MYSQL_PASSWORD=dCV3szSGNA
```

### Verify Volume Mount

```bash
kubectl exec $POD -- ls /var/lib/mysql
```

**Purpose**: Confirm the persistent storage is mounted correctly.

### Verify Service Configuration

```bash
kubectl describe svc mysql
```

**Purpose**: Confirm the NodePort and selector settings.

---

## 📚 Quick Command Reference

```bash
# Apply configuration
kubectl apply -f /home/thor/mysql_deployment.yml

# Verify resources
kubectl get pv,pvc
kubectl get svc mysql
kubectl get deploy mysql-deployment
kubectl get pod -l app=mysql-app -o wide

# Test MySQL health
POD=$(kubectl get pod -l app=mysql-app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- mysqladmin -u kodekloud_gem -pdCV3szSGNA status
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running

**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`. **Solution**: Check pod events and logs.

```bash
kubectl describe pod -l app=mysql-app
kubectl logs -l app=mysql-app
```

### Issue 2: PersistentVolumeClaim Not Bound

**Symptoms**: PVC is in `Pending` state. **Solution**: Verify PersistentVolume configuration and storage class.

```bash
kubectl describe pv mysql-pv
kubectl describe pvc mysql-pv-claim
```

### Issue 3: Incorrect NodePort

**Symptoms**: `mysql` service uses the wrong port. **Solution**: Patch the service.

```bash
kubectl patch svc mysql -p '{"spec":{"ports":[{"port":3306,"targetPort":3306,"nodePort":30011}]}}'
```

### Issue 4: Environment Variables Not Set

**Symptoms**: MySQL container fails due to missing environment variables. **Solution**: Verify secret references.

```bash
kubectl get secret mysql-root-pass mysql-user-pass mysql-db-url
kubectl exec $POD -- env | grep MYSQL
```

### Issue 5: Volume Mount Failure

**Symptoms**: MySQL data not persisting. **Solution**: Verify volume mount and PVC binding.

```bash
kubectl exec $POD -- ls /var/lib/mysql
kubectl describe pvc mysql-pv-claim
```

---

## 💡 Additional Tips

- **Pod Logs**: Check for runtime issues.

  ```bash
  kubectl logs -l app=mysql-app
  ```
- **Dry Run**: Test configuration before applying.

  ```bash
  kubectl apply -f /home/thor/mysql_deployment.yml --dry-run=client
  ```
- **MySQL Health**: Verify MySQL container status.

  ```bash
  kubectl exec $POD -- mysqladmin -u kodekloud_gem -pdCV3szSGNA status
  ```
- **NodePort Range**: Ensure 30011 is within 30000-32767.

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:

1. **Syntax Errors**: Incorrect `apiVersion` for PersistentVolume and Deployment.
2. **Indentation Issues**: Misaligned `hostPath`, `secretKeyRef`, and `volumes` sections.
3. **Field Errors**: Incorrect `images` field and `persistentVolumeReclaimPolicy` format.
4. **Storage Units**: Using `MB` instead of `Mi` for storage.
5. **Selector Misconfiguration**: Incorrect placement of `tier` in `matchLabels`.

### 💡 Solution Approach:

1. **Corrected Syntax**: Updated `apiVersion` to `v1` for PersistentVolume and `apps/v1` for Deployment.
2. **Fixed Indentation**: Ensured proper alignment for `hostPath`, `secretKeyRef`, and `volumes`.
3. **Corrected Fields**: Changed `images` to `image` and fixed `persistentVolumeReclaimPolicy`.
4. **Updated Storage Units**: Changed `250MB` to `250Mi`.
5. **Fixed Selector**: Moved `tier: mysql` under `matchLabels`.
6. **Verification**: Confirmed resources and MySQL health.

### 🎯 Key Success Factors:

- **PersistentVolume**: Correctly configured with `Retain` policy
- **PersistentVolumeClaim**: Bound to the correct volume
- **Service**: Exposed on NodePort 30011
- **Deployment**: Runs MySQL with correct environment variables
- **Volumes**: Properly mounted for persistent storage

### ⚠️ Critical Configuration Details:

- **PersistentVolume**: `mysql-pv`
- **PersistentVolumeClaim**: `mysql-pv-claim`
- **Service**: `mysql` (NodePort 30011)
- **Deployment**: `mysql-deployment`
- **Volume Path**: `/var/lib/mysql`

### 🔒 Kubernetes Benefits:

- **Persistence**: PersistentVolume ensures data retention
- **Scalability**: Deployment supports controlled updates
- **Accessibility**: NodePort enables external MySQL access
- **Reliability**: Kubernetes manages pod health

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates in production. 🔐 **Security**: Implement network policies to restrict MySQL access. 📊 **Monitoring**: Monitor MySQL health and performance. 🛡️ **Scalability**: Consider a separate MySQL cluster for high availability.

---

## 📖 Learning Outcomes

**Key Concepts Mastered**:

- **PersistentVolume and PersistentVolumeClaim**: Managing storage in Kubernetes
- **Service**: Exposing MySQL via NodePort
- **Deployment**: Running MySQL with environment variables
- **Environment Variables**: Sourcing from secrets
- **Troubleshooting**: Fixing YAML syntax and indentation errors

**Kubernetes Features Used**:

- `v1 PersistentVolume` for storage
- `v1 PersistentVolumeClaim` for storage requests
- `v1 Service` for NodePort exposure
- `apps/v1 Deployment` for MySQL container
- Secret references for environment variables

---

## 🎯 Task Completion Summary

✅ **Successfully Completed**:

- **PersistentVolume** - Created `mysql-pv` with 250Mi storage
- **PersistentVolumeClaim** - Created `mysql-pv-claim` bound to `mysql-pv`
- **Service** - Created `mysql` on NodePort 30011
- **Deployment** - Created `mysql-deployment` with MySQL 5.6
- **Verification** - Confirmed resources and MySQL health

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Deployment Foundation**: The MySQL deployment is fixed, applied, and running correctly, ready for integration with other applications by the Nautilus DevOps team.

### 🔮 Deployment Ready for:

- **Integration**: Connect with applications needing MySQL
- **Security**: Add network policies and secret encryption
- **Monitoring**: Implement health checks and logging
- **Scaling**: Increase replicas or use a dedicated MySQL cluster 