# 🌟 Task 1 - Deploy Redis on Kubernetes Cluster

## 📌 Task Description

The Nautilus application development team identified performance issues with one of their applications deployed in a Kubernetes cluster. To address this, they plan to implement an in-memory caching utility for their database service, choosing Redis as the solution. The team wants to deploy Redis initially for testing before moving to production. Your task is to create a Redis deployment with the specified configurations and ensure it is running correctly.

### Requirements:
- **ConfigMap**: Create a ConfigMap named `my-redis-config` with `maxmemory 2mb` in `redis-config`.
- **Deployment**: Create `redis-deployment` using the image `redis:alpine`, with the container named `redis-container` and exactly 1 replica.
- **Resources**: The container should request 1 CPU.
- **Volumes**:
  - An empty directory volume named `data` mounted at `/redis-master-data`.
  - A ConfigMap volume named `redis-config` mounted at `/redis-master`.
- **Port**: The container should expose port 6379.
- **Verification**: Ensure `redis-deployment` is up and running.
- **Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Create the ConfigMap and deployment configurations, apply them, and verify that the Redis deployment is operational.

---

## 🔧 Infrastructure Overview

**Target Environment**: Kubernetes Cluster  
**Resources**:
- ConfigMap: `my-redis-config` for Redis configuration
- Deployment: `redis-deployment` for the Redis application
- Namespace: Default

**Working Directory**: Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **ConfigMap**: Stores Redis configuration (`maxmemory 2mb`)
- **Deployment**: Runs the Redis application with the `redis:alpine` image
- **Volumes**: Provides persistent storage and configuration for Redis
- **Application**: Redis in-memory caching service accessible on port 6379

### 🎯 Implementation Strategy
1. Create a ConfigMap with the specified Redis configuration.
2. Create a deployment using the `redis:alpine` image with the required settings.
3. Configure the container to request 1 CPU and expose port 6379.
4. Mount the specified volumes (emptyDir and ConfigMap).
5. Apply the configurations using `kubectl`.
6. Verify the deployment, pod status, and ConfigMap mount.

---

## 🚨 Potential Errors to Avoid

1. **Incorrect ConfigMap Key**:
   **Issue**: Using a wrong key or format for `maxmemory` in the ConfigMap.
   **Fix**: Ensure the ConfigMap key is `redis-config` with the value `maxmemory 2mb`.

2. **Volume Mount Misconfiguration**:
   **Issue**: Incorrect mount paths or volume names.
   **Fix**: Verify mount paths (`/redis-master-data`, `/redis-master`) and volume names (`data`, `redis-config`).

3. **Image or Resource Misconfiguration**:
   **Issue**: Using a different Redis image or incorrect CPU request.
   **Fix**: Use `redis:alpine` and set `resources.requests.cpu` to `1`.

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Configuration File

#### Redis ConfigMap and Deployment (`redis-deployment-task1.yaml`)
Create or edit a file named `redis-deployment-task1.yaml`:

```yaml
# ConfigMap for Redis Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
  namespace: default
data:
  redis-config: |
    maxmemory 2mb

---
# Redis Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        resources:
          requests:
            cpu: "1"
        ports:
        - containerPort: 6379
          name: redis-port
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
```

**Configuration Breakdown**:
- **ConfigMap**:
  - **Name**: `my-redis-config`
  - **Namespace**: `default`
  - **Data**: `redis-config` with `maxmemory 2mb`
- **Deployment**:
  - **Name**: `redis-deployment`
  - **Namespace**: `default`
  - **Labels**: `app: redis`
  - **Replicas**: 1
  - **Selector**: `matchLabels` with `app: redis`
  - **Container**: `redis-container` with image `redis:alpine`
  - **Resources**: Requests 1 CPU
  - **Ports**: Exposes port 6379 (named `redis-port`)
  - **Volumes**:
    - `data`: Empty directory mounted at `/redis-master-data`
    - `redis-config`: ConfigMap `my-redis-config` mounted at `/redis-master`

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configurations
```bash
kubectl apply -f redis-deployment-task1.yaml
```

**Purpose**: Create the ConfigMap and deployment in the Kubernetes cluster.

**Expected Output**:
```
configmap/my-redis-config created
deployment.apps/redis-deployment created
```

---

### Step 4: Verify Resources
```bash
kubectl get configmap my-redis-config
```

**Expected Output**:
```
NAME              DATA   AGE
my-redis-config   1      2m
```

```bash
kubectl get deployments.apps redis-deployment
```

**Expected Output**:
```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           2m
```

```bash
kubectl get pod -l app=redis
```

**Expected Output**:
```
NAME                              READY   STATUS    RESTARTS   AGE
redis-deployment-68fbd4467-dr7d7   1/1     Running   0          2m
```

```bash
kubectl describe deployment redis-deployment
```

**Expected Output (Excerpt)**:
```
Name:                   redis-deployment
Namespace:              default
Selector:               app=redis
Replicas:               1 desired | 1 updated | 1 total | 1 available
Pod Template:
  Containers:
   redis-container:
    Image:        redis:alpine
    Port:         6379/TCP
    Requests:
      cpu:        1
    Volume Mounts:
      /redis-master-data from data (rw)
      /redis-master from redis-config (rw)
  Volumes:
   data:
    Type:       EmptyDir
   redis-config:
    Type:       ConfigMap
    Name:       my-redis-config
```

---

### Step 5: Verify ConfigMap Mount
```bash
POD=$(kubectl get pod -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- cat /redis-master/redis-config
```

**Expected Output**:
```
maxmemory 2mb
```

**Purpose**: Confirm the ConfigMap is correctly mounted and contains the specified configuration.

---

### Step 6: Verify Redis Port
```bash
kubectl exec $POD -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN
```

**Purpose**: Confirm Redis is listening on port 6379.

---

## 🔍 Code Analysis

### Redis ConfigMap and Deployment (`redis-deployment-task1.yaml`)
```yaml
# ConfigMap for Redis Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
  namespace: default
data:
  redis-config: |
    maxmemory 2mb

---
# Redis Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        resources:
          requests:
            cpu: "1"
        ports:
        - containerPort: 6379
          name: redis-port
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **ConfigMap** | my-redis-config | redis-config: maxmemory 2mb | Stores Redis memory configuration |
| **Deployment** | redis-deployment | replicas: 1, app: redis | Runs Redis application |
| **Container** | redis-container | image: redis:alpine | Redis container |
| **Resources** | requests | cpu: 1 | CPU request for container |
| **Container Port** | redis-port | 6379 | Default Redis port |
| **Volume (data)** | data | emptyDir | Mounted at /redis-master-data |
| **Volume (redis-config)** | redis-config | ConfigMap: my-redis-config | Mounted at /redis-master |

---

## ✅ Verification Steps

### Step 1: Verify ConfigMap
```bash
kubectl get configmap my-redis-config
```

**Expected Output**:
```
NAME              DATA   AGE
my-redis-config   1      2m
```

### Step 2: Verify Deployment
```bash
kubectl get deployments.apps redis-deployment
```

**Expected Output**:
```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           2m
```

### Step 3: Verify Pods
```bash
kubectl get pod -l app=redis
```

**Expected Output**:
```
NAME                              READY   STATUS    RESTARTS   AGE
redis-deployment-68fbd4467-dr7d7   1/1     Running   0          2m
```

### Step 4: Verify ConfigMap Mount
```bash
POD=$(kubectl get pod -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- cat /redis-master/redis-config
```

**Expected Output**:
```
maxmemory 2mb
```

### Step 5: Verify Redis Port
```bash
kubectl exec $POD -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN
```

---

## 🧪 Testing

### Verify Redis Functionality
```bash
POD=$(kubectl get pod -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping
```

**Expected Output**:
```
PONG
```

**Purpose**: Confirm the Redis server is operational.

### Verify ConfigMap Content
```bash
kubectl exec -it $POD -- cat /redis-master/redis-config
```

**Purpose**: Ensure the ConfigMap is correctly mounted.

---

## 📚 Quick Command Reference
```bash
# Apply configurations
kubectl apply -f redis-deployment-task1.yaml

# Verify resources
kubectl get configmap my-redis-config
kubectl get deployments.apps redis-deployment
kubectl get pod -l app=redis

# Verify ConfigMap mount
POD=$(kubectl get pod -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- cat /redis-master/redis-config

# Verify Redis port
kubectl exec $POD -- netstat -tuln

# Test Redis functionality
kubectl exec -it $POD -- redis-cli ping
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.  
**Solution**: Check pod events and logs.
```bash
kubectl describe pod -l app=redis
kubectl logs -l app=redis
```

### Issue 2: ConfigMap Not Mounted
**Symptoms**: `cat /redis-master/redis-config` returns empty or errors.  
**Solution**: Verify ConfigMap and volume mount configuration.
```bash
kubectl describe configmap my-redis-config
kubectl describe deployment redis-deployment
```

### Issue 3: Image Pull Failure
**Symptoms**: Pod fails with `ImagePullBackOff`.  
**Solution**: Verify the image name (`redis:alpine`).
```bash
kubectl describe pod -l app=redis
```

### Issue 4: Incorrect CPU Request
**Symptoms**: Deployment fails due to resource constraints.  
**Solution**: Confirm CPU request is set to `1`.
```bash
kubectl describe deployment redis-deployment
```

---

## 💡 Additional Tips
- **Pod Logs**: Check for runtime issues.
  ```bash
  kubectl logs -l app=redis
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f redis-deployment-task1.yaml --dry-run=client
  ```
- **Redis CLI**: Test Redis commands.
  ```bash
  POD=$(kubectl get pod -l app=redis -o jsonpath='{.items[0].metadata.name}')
  kubectl exec -it $POD -- redis-cli
  ```
- **Volume Verification**: Confirm volume mounts.
  ```bash
  kubectl exec $POD -- ls /redis-master /redis-master-data
  ```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **ConfigMap Configuration**: Ensuring the correct key (`redis-config`) and value (`maxmemory 2mb`).
2. **Volume Mounts**: Correctly mounting the emptyDir and ConfigMap volumes.
3. **Resource Requests**: Setting the CPU request to exactly 1.

### 💡 Solution Approach:
1. **ConfigMap Creation**: Defined `my-redis-config` with the correct configuration.
2. **Deployment Setup**: Configured `redis-deployment` with `redis:alpine`, 1 replica, CPU request, and port 6379.
3. **Volume Mounts**: Added emptyDir and ConfigMap volumes with correct mount paths.
4. **Verification**: Confirmed deployment status, pod health, and ConfigMap mount.

### 🎯 Key Success Factors:
- **ConfigMap**: Correctly stores `maxmemory 2mb`.
- **Deployment**: Runs Redis with the specified image and resources.
- **Volumes**: Correctly mounted for data and configuration.
- **Redis**: Operational on port 6379.

### ⚠️ Critical Configuration Details:
- **ConfigMap**: `my-redis-config`
- **Deployment**: `redis-deployment`
- **Image**: `redis:alpine`
- **Container**: `redis-container`
- **CPU Request**: `1`
- **Port**: `6379`
- **Volumes**: `data` (emptyDir), `redis-config` (ConfigMap)

### 🔒 Kubernetes Benefits:
- **Scalability**: Deployment supports replicas for future scaling.
- **Flexibility**: ConfigMap allows easy configuration updates.
- **Reliability**: Kubernetes manages pod health and restarts.

---

## ⚠️ Important Production Notes
🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.  
🔐 **Security**: Implement network policies to restrict Redis access.  
📊 **Monitoring**: Monitor Redis performance and memory usage.  
🛡️ **Persistence**: Consider persistent volumes for production use.

---

## 📖 Learning Outcomes
**Key Concepts Mastered**:
- **ConfigMap**: Managing configuration data for applications.
- **Deployment**: Managing Redis pods with specific resources.
- **Volumes**: Using emptyDir and ConfigMap volumes.
- **Verification**: Ensuring resource creation and functionality.

**Kubernetes Features Used**:
- `v1 ConfigMap` for Redis configuration.
- `apps/v1 Deployment` for Redis application.

---

## 🎯 Task Completion Summary
✅ **Successfully Completed**:
- **ConfigMap** - Created `my-redis-config` with `maxmemory 2mb`.
- **Deployment** - Created `redis-deployment` with `redis:alpine`, 1 replica, 1 CPU, and port 6379.
- **Volumes** - Mounted emptyDir at `/redis-master-data` and ConfigMap at `/redis-master`.
- **Verification** - Confirmed deployment is running and ConfigMap is correctly mounted.

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Application Foundation**: The Redis deployment is up and running, ready for testing by the Nautilus DevOps team, with a solid foundation for future production enhancements.

### 🔮 Application Ready for:
- **Testing**: Validate caching performance.
- **Scaling**: Increase replicas for high availability.
- **Security**: Add network policies and authentication.
- **Monitoring**: Implement health checks and metrics collection.