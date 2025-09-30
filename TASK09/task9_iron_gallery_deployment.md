# 🌟 Task 9 - Deploy Iron Gallery App on Kubernetes Cluster

## 📌 Task Description

The Nautilus DevOps team is deploying a customized Iron Gallery application on a Kubernetes cluster. The application consists of a front-end (Iron Gallery) and a database (Iron DB). The task involves creating a namespace, two deployments, and two services to deploy the application, ensuring proper configuration for storage, environment variables, and service exposure.

### Requirements:
- **Namespace**: Create `iron-namespace-xfusion`.
- **Iron Gallery Deployment** (`iron-gallery-deployment-xfusion`):
  - Namespace: `iron-namespace-xfusion`
  - Labels: `run: iron-gallery`
  - Replicas: 1
  - Selector: `matchLabels` with `run: iron-gallery`
  - Template labels: `run: iron-gallery`
  - Container: `iron-gallery-container-xfusion` using `kodekloud/irongallery:2.0`
  - Resources: Memory limit `100Mi`, CPU limit `50m`
  - VolumeMounts:
    - Name: `config`, mountPath: `/usr/share/nginx/html/data`
    - Name: `images`, mountPath: `/usr/share/nginx/html/uploads`
  - Volumes:
    - Name: `config`, type: `emptyDir`
    - Name: `images`, type: `emptyDir`
- **Iron DB Deployment** (`iron-db-deployment-xfusion`):
  - Namespace: `iron-namespace-xfusion`
  - Labels: `db: mariadb`
  - Replicas: 1
  - Selector: `matchLabels` with `db: mariadb`
  - Template labels: `db: mariadb`
  - Container: `iron-db-container-xfusion` using `kodekloud/irondb:2.0`
  - Environment Variables:
    - `MYSQL_DATABASE`: `database_web`
    - `MYSQL_ROOT_PASSWORD`: Complex password (e.g., `RootPass123!`)
    - `MYSQL_USER`: Custom user (e.g., `customuser`)
    - `MYSQL_PASSWORD`: Complex password (e.g., `CustomPass456!`)
  - VolumeMount: Name: `db`, mountPath: `/var/lib/mysql`
  - Volume: Name: `db`, type: `emptyDir`
- **Iron DB Service** (`iron-db-service-xfusion`):
  - Namespace: `iron-namespace-xfusion`
  - Selector: `db: mariadb`
  - Protocol: TCP
  - Port: 3306
  - TargetPort: 3306
  - Type: ClusterIP
- **Iron Gallery Service** (`iron-gallery-service-xfusion`):
  - Namespace: `iron-namespace-xfusion`
  - Selector: `run: iron-gallery`
  - Protocol: TCP
  - Port: 80
  - TargetPort: 80
  - NodePort: 32678
  - Type: NodePort
- **Verification**: Ensure the application installation page is accessible via the app button (no database connection required for now).

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. No database connection to the front-end is needed at this stage.

👉 **Your task**: Deploy the Iron Gallery application and database on the Kubernetes cluster, ensuring all components are correctly configured and the application installation page is accessible.

---

## 🔧 Infrastructure Overview

**Target Environment**: Kubernetes Cluster
**Resources**:
- Namespace for isolation
- Deployments for Iron Gallery and Iron DB
- Services for exposing the application and database
- Volumes for temporary storage
- Environment variables for database configuration

**Working Directory**: Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Namespace**: Isolates resources for the Iron Gallery app
- **Iron Gallery Deployment**: Runs the front-end application
- **Iron DB Deployment**: Runs the MariaDB database
- **Services**: Expose the application (NodePort) and database (ClusterIP)
- **Volumes**: Provide temporary storage for the application and database
- **Environment Variables**: Configure the database securely

### 🎯 Implementation Strategy
1. Create the namespace `iron-namespace-xfusion`
2. Create and apply the Iron Gallery deployment with correct labels, resources, and volumes
3. Create and apply the Iron DB deployment with environment variables and volume
4. Create and apply services for Iron Gallery and Iron DB
5. Verify resource creation and application accessibility
6. Fix errors in the provided YAML configurations

---



---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Namespace
```bash
kubectl create namespace iron-namespace-xfusion
kubectl config set-context --current --namespace=iron-namespace-xfusion
```

**Purpose**: Create the namespace and set it as the default for subsequent commands.

**Expected Output**:
```
namespace/iron-namespace-xfusion created
context "current-context" modified
```

---

### Step 3: Create Configuration Files

#### Iron Gallery Deployment (`iron-gallery-deployment-xfusion.yaml`)
Create a file named `iron-gallery-deployment-xfusion.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-xfusion
  namespace: iron-namespace-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-xfusion
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads
      volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}
```

**Configuration Breakdown**:
- **Deployment**: `iron-gallery-deployment-xfusion`
- **Labels**: `run: iron-gallery`
- **Replicas**: 1
- **Selector**: `matchLabels` with `run: iron-gallery`
- **Container**: `iron-gallery-container-xfusion` with image `kodekloud/irongallery:2.0`
- **Resources**: Memory limit `100Mi`, CPU limit `50m`
- **Ports**: Exposes port 80 for HTTP traffic
- **VolumeMounts**: `config` at `/usr/share/nginx/html/data`, `images` at `/usr/share/nginx/html/uploads`
- **Volumes**: Two `emptyDir` volumes (`config` and `images`)

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

#### Iron DB Deployment (`iron-db-deployment-xfusion.yaml`)
Create a file named `iron-db-deployment-xfusion.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-xfusion
  namespace: iron-namespace-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-xfusion
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: database_web
            - name: MYSQL_ROOT_PASSWORD
              value: RootPass123!
            - name: MYSQL_USER
              value: customuser
            - name: MYSQL_PASSWORD
              value: CustomPass456!
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}
```

**Configuration Breakdown**:
- **Deployment**: `iron-db-deployment-xfusion`
- **Labels**: `db: mariadb`
- **Replicas**: 1
- **Selector**: `matchLabels` with `db: mariadb`
- **Container**: `iron-db-container-xfusion` with image `kodekloud/irondb:2.0`
- **Environment Variables**:
  - `MYSQL_DATABASE`: `database_web`
  - `MYSQL_ROOT_PASSWORD`: `RootPass123!`
  - `MYSQL_USER`: `customuser`
  - `MYSQL_PASSWORD`: `CustomPass456!`
- **Ports**: Exposes port 3306 for MySQL
- **VolumeMount**: `db` at `/var/lib/mysql`
- **Volume**: `emptyDir` named `db`

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

#### Services (`iron-services-xfusion.yaml`)
Create a file named `iron-services-xfusion.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
  type: NodePort
```

**Configuration Breakdown**:
- **Iron DB Service** (`iron-db-service-xfusion`):
  - Selector: `db: mariadb`
  - Port: 3306, TargetPort: 3306
  - Type: ClusterIP
- **Iron Gallery Service** (`iron-gallery-service-xfusion`):
  - Selector: `run: iron-gallery`
  - Port: 80, TargetPort: 80, NodePort: 32678
  - Type: NodePort

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 4: Apply Configurations
```bash
kubectl apply -f iron-gallery-deployment-xfusion.yaml
kubectl apply -f iron-db-deployment-xfusion.yaml
kubectl apply -f iron-services-xfusion.yaml
```

**Purpose**: Deploy the namespace, deployments, and services.

**Expected Output**:
```
deployment.apps/iron-gallery-deployment-xfusion created
deployment.apps/iron-db-deployment-xfusion created
service/iron-db-service-xfusion created
service/iron-gallery-service-xfusion created
```

---

### Step 5: Verify Resources
```bash
kubectl get namespace iron-namespace-xfusion
```

**Expected Output**:
```
NAME                    STATUS   AGE
iron-namespace-xfusion   Active   2m
```

```bash
kubectl get deploy -n iron-namespace-xfusion
```

**Expected Output**:
```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
iron-gallery-deployment-xfusion   1/1     1            1           2m
iron-db-deployment-xfusion        1/1     1            1           2m
```

```bash
kubectl get pod -n iron-namespace-xfusion
```

**Expected Output**:
```
NAME                                            READY   STATUS    RESTARTS   AGE
iron-gallery-deployment-xfusion-abc123-xyz123    1/1     Running   0          2m
iron-db-deployment-xfusion-def456-uvw789         1/1     Running   0          2m
```

```bash
kubectl get svc -n iron-namespace-xfusion
```

**Expected Output**:
```
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
iron-db-service-xfusion       ClusterIP   10.96.123.456   <none>        3306/TCP         2m
iron-gallery-service-xfusion   NodePort    10.96.123.457   <none>        80:32678/TCP     2m
```

---

### Step 6: Verify Application Accessibility
**Access the Application**:
- Click the **App** button in the lab interface and this page will appear ![screenshot](<Lychee -.png>)

- Alternatively, use the cluster’s node IP:(optional)
```bash
curl http://<node-ip>:32678
```

**Expected Output**: The Iron Gallery installation page should appear, indicating the application is running.

**Verification**: Confirm the installation page is accessible via the app button.

---

## 🔍 Code Analysis

### Iron Gallery Deployment (`iron-gallery-deployment-xfusion.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-xfusion
  namespace: iron-namespace-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-xfusion
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads
      volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}
```

### Iron DB Deployment (`iron-db-deployment-xfusion.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-xfusion
  namespace: iron-namespace-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-xfusion
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: database_web
            - name: MYSQL_ROOT_PASSWORD
              value: RootPass123!
            - name: MYSQL_USER
              value: customuser
            - name: MYSQL_PASSWORD
              value: CustomPass456!
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}
```

### Services (`iron-services-xfusion.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
  type: NodePort
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **Namespace** | iron-namespace-xfusion | - | Isolates Iron Gallery resources |
| **Iron Gallery Deployment** | iron-gallery-deployment-xfusion | replicas: 1, run: iron-gallery | Runs front-end application |
| **Iron Gallery Container** | iron-gallery-container-xfusion | image: kodekloud/irongallery:2.0 | Nginx-based front-end |
| **Iron Gallery Resources** | limits | memory: 100Mi, cpu: 50m | Resource constraints |
| **Iron Gallery VolumeMounts** | config, images | /usr/share/nginx/html/data, /usr/share/nginx/html/uploads | Temporary storage paths |
| **Iron DB Deployment** | iron-db-deployment-xfusion | replicas: 1, db: mariadb | Runs MariaDB database |
| **Iron DB Container** | iron-db-container-xfusion | image: kodekloud/irondb:2.0 | MariaDB database |
| **Iron DB Environment** | MYSQL_DATABASE, MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD | database_web, RootPass123!, customuser, CustomPass456! | Database configuration |
| **Iron DB VolumeMount** | db | /var/lib/mysql | Temporary storage for DB |
| **Iron DB Service** | iron-db-service-xfusion | type: ClusterIP, port: 3306 | Exposes MariaDB internally |
| **Iron Gallery Service** | iron-gallery-service-xfusion | type: NodePort, port: 80, nodePort: 32678 | Exposes front-end externally |

---

## ✅ Verification Steps

### Step 1: Verify Namespace
```bash
kubectl get namespace iron-namespace-xfusion
```

**Expected Output**:
```
NAME                    STATUS   AGE
iron-namespace-xfusion   Active   2m
```

### Step 2: Verify Deployments
```bash
kubectl get deploy -n iron-namespace-xfusion
```

**Expected Output**:
```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
iron-gallery-deployment-xfusion   1/1     1            1           2m
iron-db-deployment-xfusion        1/1     1            1           2m
```

### Step 3: Verify Pods
```bash
kubectl get pod -n iron-namespace-xfusion
```

**Expected Output**:
```
NAME                                            READY   STATUS    RESTARTS   AGE
iron-gallery-deployment-xfusion-abc123-xyz123    1/1     Running   0          2m
iron-db-deployment-xfusion-def456-uvw789         1/1     Running   0          2m
```

### Step 4: Verify Services
```bash
kubectl get svc -n iron-namespace-xfusion
```

**Expected Output**:
```
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
iron-db-service-xfusion       ClusterIP   10.96.123.456   <none>        3306/TCP         2m
iron-gallery-service-xfusion   NodePort    10.96.123.457   <none>        80:32678/TCP     2m
```

### Step 5: Verify Application
**Access the Application**:
- Click the **App** button in the lab interface.
- Alternatively, test with:
```bash
curl http://<node-ip>:32678
```

**Expected Output**: The Iron Gallery installation page should be displayed.

---

## 🧪 Testing

### Verify Iron Gallery Container
```bash
POD=$(kubectl get pod -n iron-namespace-xfusion -l run=iron-gallery -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n iron-namespace-xfusion $POD -- ls /usr/share/nginx/html
```

**Purpose**: Confirm the `config` and `images` directories are mounted.

### Verify Iron DB Environment Variables
```bash
POD=$(kubectl get pod -n iron-namespace-xfusion -l db=mariadb -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n iron-namespace-xfusion $POD -- env | grep MYSQL
```

**Expected Output**:
```
MYSQL_DATABASE=database_web
MYSQL_ROOT_PASSWORD=RootPass123!
MYSQL_USER=customuser
MYSQL_PASSWORD=CustomPass456!
```

### Verify Iron DB Health
```bash
kubectl exec -n iron-namespace-xfusion $POD -- mysqladmin -u customuser -pCustomPass456! status
```

**Expected Output**:
```
Uptime: 120  Threads: 1  Questions: 1  Slow queries: 0  Opens: 1  Flush tables: 1  Open tables: 0  Queries per second avg: 0.008
```

### Verify Service Configuration
```bash
kubectl describe svc iron-gallery-service-xfusion -n iron-namespace-xfusion
kubectl describe svc iron-db-service-xfusion -n iron-namespace-xfusion
```

**Purpose**: Confirm correct selectors and ports.

---

## 📚 Quick Command Reference
```bash
# Create namespace
kubectl create namespace iron-namespace-xfusion
kubectl config set-context --current --namespace=iron-namespace-xfusion

# Apply configurations
kubectl apply -f iron-gallery-deployment-xfusion.yaml
kubectl apply -f iron-db-deployment-xfusion.yaml
kubectl apply -f iron-services-xfusion.yaml

# Verify resources
kubectl get namespace iron-namespace-xfusion
kubectl get deploy -n iron-namespace-xfusion
kubectl get pod -n iron-namespace-xfusion
kubectl get svc -n iron-namespace-xfusion

# Test application
curl http://<node-ip>:32678
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod -n iron-namespace-xfusion -l run=iron-gallery
kubectl describe pod -n iron-namespace-xfusion -l db=mariadb
kubectl logs -n iron-namespace-xfusion -l run=iron-gallery
kubectl logs -n iron-namespace-xfusion -l db=mariadb
```

### Issue 2: Service Not Accessible
**Symptoms**: `iron-gallery-service-xfusion` not responding on port 32678.
**Solution**: Verify service and pod labels.
```bash
kubectl describe svc iron-gallery-service-xfusion -n iron-namespace-xfusion
kubectl get pod -n iron-namespace-xfusion -l run=iron-gallery -o wide
```

### Issue 3: Incorrect NodePort
**Symptoms**: `iron-gallery-service-xfusion` uses wrong port.
**Solution**: Patch the service.
```bash
kubectl patch svc iron-gallery-service-xfusion -n iron-namespace-xfusion -p '{"spec":{"ports":[{"port":80,"targetPort":80,"nodePort":32678}]}}'
```

### Issue 4: Volume Mount Failure
**Symptoms**: Application or database data not persisting.
**Solution**: Verify volume mounts.
```bash
POD=$(kubectl get pod -n iron-namespace-xfusion -l run=iron-gallery -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n iron-namespace-xfusion $POD -- ls /usr/share/nginx/html
POD=$(kubectl get pod -n iron-namespace-xfusion -l db=mariadb -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n iron-namespace-xfusion $POD -- ls /var/lib/mysql
```

### Issue 5: Environment Variables Not Set
**Symptoms**: Iron DB container fails due to missing environment variables.
**Solution**: Verify environment variables.
```bash
POD=$(kubectl get pod -n iron-namespace-xfusion -l db=mariadb -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n iron-namespace-xfusion $POD -- env | grep MYSQL
```

---

## 💡 Additional Tips
- **Pod Logs**: Check for runtime issues.
  ```bash
  kubectl logs -n iron-namespace-xfusion -l run=iron-gallery
  kubectl logs -n iron-namespace-xfusion -l db=mariadb
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f iron-gallery-deployment-xfusion.yaml --dry-run=client
  kubectl apply -f iron-db-deployment-xfusion.yaml --dry-run=client
  kubectl apply -f iron-services-xfusion.yaml --dry-run=client
  ```
- **MySQL Health**: Verify Iron DB container status.
  ```bash
  POD=$(kubectl get pod -n iron-namespace-xfusion -l db=mariadb -o jsonpath='{.items[0].metadata.name}')
  kubectl exec -n iron-namespace-xfusion $POD -- mysqladmin -u customuser -pCustomPass456! status
  ```
- **NodePort Range**: Ensure 32678 is within 30000-32767.

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Namespace Isolation**: Ensuring all resources are created in `iron-namespace-xfusion`.
2. **Volume Configuration**: Correctly setting up `emptyDir` volumes for both deployments.
3. **Service Exposure**: Configuring NodePort (32678) and ClusterIP services correctly.
4. **Environment Variables**: Setting secure passwords for Iron DB.
5. **Error Fixes**: Correcting missing volume name and port definition in the Iron Gallery deployment.

### 💡 Solution Approach:
1. **Namespace**: Created `iron-namespace-xfusion` and set it as the default context.
2. **Deployments**: Configured `iron-gallery-deployment-xfusion` and `iron-db-deployment-xfusion` with correct labels, replicas, and volumes.
3. **Services**: Set up `iron-gallery-service-xfusion` (NodePort 32678) and `iron-db-service-xfusion` (ClusterIP 3306).
4. **Error Fixes**: Added missing `name: images` for the volume and `containerPort: 80` for the Iron Gallery container.
5. **Verification**: Confirmed resources and application installation page accessibility.

### 🎯 Key Success Factors:
- **Namespace**: Resources isolated in `iron-namespace-xfusion`
- **Deployments**: Correctly configured with labels and resources
- **Services**: Properly expose the application and database
- **Volumes**: Correctly mounted for temporary storage
- **Application**: Installation page accessible

### ⚠️ Critical Configuration Details:
- **Namespace**: `iron-namespace-xfusion`
- **Deployments**: `iron-gallery-deployment-xfusion`, `iron-db-deployment-xfusion`
- **Services**: `iron-gallery-service-xfusion` (NodePort 32678), `iron-db-service-xfusion` (ClusterIP 3306)
- **Volumes**: `config`, `images`, `db` as `emptyDir`

### 🔒 Kubernetes Benefits:
- **Isolation**: Namespace ensures resource separation
- **Scalability**: Deployments support replicas
- **Accessibility**: NodePort enables external access to the application
- **Reliability**: Kubernetes manages pod health

---

## ⚠️ Important Production Notes
🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.
🔐 **Security**: Implement network policies for database access.
📊 **Monitoring**: Monitor application and database health.
🛡️ **Scalability**: Consider persistent storage for production.

---

## 📖 Learning Outcomes
**Key Concepts Mastered**:
- **Namespace**: Resource isolation in Kubernetes
- **Deployments**: Managing application and database pods
- **Services**: Exposing applications via NodePort and ClusterIP
- **Volumes**: Using `emptyDir` for temporary storage
- **Environment Variables**: Configuring database credentials
- **Troubleshooting**: Fixing YAML errors

**Kubernetes Features Used**:
- `v1 Namespace` for resource isolation
- `apps/v1 Deployment` for application and database
- `v1 Service` for NodePort and ClusterIP
- `emptyDir` volumes for temporary storage

---

## 🎯 Task Completion Summary
✅ **Successfully Completed**:
- **Namespace** - Created `iron-namespace-xfusion`
- **Deployments** - Created `iron-gallery-deployment-xfusion` and `iron-db-deployment-xfusion`
- **Services** - Created `iron-gallery-service-xfusion` (NodePort 32678) and `iron-db-service-xfusion` (ClusterIP 3306)
- **Verification** - Confirmed resources and application installation page

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Application Foundation**: The Iron Gallery application and database are deployed, with the installation page accessible via NodePort 32678, ready for further configuration by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Database Integration**: Connect Iron Gallery to Iron DB
- **Security**: Add network policies and secret management
- **Monitoring**: Implement health checks and logging
- **Content Updates**: Complete the application setup