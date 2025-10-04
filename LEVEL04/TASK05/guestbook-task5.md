# 🌟 Task 5 - Deploy Guestbook Application on Kubernetes Cluster

## 📌 Task Description

The Nautilus Application Development team has completed development of a guestbook application to manage guest/visitor entries, and it is ready for deployment on a Kubernetes cluster. The application uses a Redis backend (master and slave) and a PHP-based frontend. The DevOps team has finalized the infrastructure requirements, including deployments and services for both tiers. Your task is to create the required Kubernetes resources and ensure the application is accessible.

### Requirements:
**Back-End Tier**:
1. **Redis Master Deployment**: Create `redis-master` with:
   - 1 replica
   - Container named `master-redis-datacenter`, using `redis` image
   - Resource requests: 100m CPU, 100Mi memory
   - Container port: 6379 (Redis default)
2. **Redis Master Service**: Create `redis-master` with port and targetPort 6379.
3. **Redis Slave Deployment**: Create `redis-slave` with:
   - 2 replicas
   - Container named `slave-redis-datacenter`, using `gcr.io/google_samples/gb-redisslave:v3`
   - Resource requests: 100m CPU, 100Mi memory
   - Environment variable `GET_HOSTS_FROM` set to `dns`
   - Container port: 6379
4. **Redis Slave Service**: Create `redis-slave` with port and targetPort 6379.

**Front-End Tier**:
1. **Frontend Deployment**: Create `frontend` with:
   - 3 replicas
   - Container named `php-redis-datacenter`, using `gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff`
   - Resource requests: 100m CPU, 100Mi memory
   - Environment variable `GET_HOSTS_FROM` set to `dns`
   - Container port: 80
2. **Frontend Service**: Create `frontend` as NodePort with:
   - Port: 80
   - TargetPort: 80
   - NodePort: 30009

**Verification**: Ensure all resources are running, and the guestbook application is accessible via the **App** button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Create the deployments and services for the Redis master, Redis slave, and frontend, apply them, and verify that the guestbook application is accessible.

---

## 🔧 Infrastructure Overview

**Target Environment**: Kubernetes Cluster  
**Resources**:
- Deployment: `redis-master` for Redis master
- Service: `redis-master` to expose Redis master
- Deployment: `redis-slave` for Redis slaves
- Service: `redis-slave` to expose Redis slaves
- Deployment: `frontend` for the guestbook frontend
- Service: `frontend` to expose the frontend via NodePort
- Namespace: Default

**Working Directory**: Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Redis Master Deployment**: Runs a single Redis master instance
- **Redis Master Service**: Exposes the master for internal communication
- **Redis Slave Deployment**: Runs two Redis slave instances for replication
- **Redis Slave Service**: Exposes slaves for load balancing
- **Frontend Deployment**: Runs the PHP-based guestbook application
- **Frontend Service**: Exposes the frontend externally via NodePort
- **Application**: Guestbook app with Redis backend and PHP frontend

### 🎯 Implementation Strategy
1. Create deployments for Redis master, Redis slave, and frontend with specified configurations.
2. Create services to expose Redis master, Redis slave, and frontend.
3. Apply the configurations using `kubectl`.
4. Verify resource creation, pod status, and service accessibility.
5. Confirm the guestbook application is accessible via the **App** button.

---

## 🚨 Potential Errors to Avoid

1. **Image Misconfiguration**:
   **Issue**: Incorrect image names or tags.
   **Fix**: Use `redis` for master, `gcr.io/google_samples/gb-redisslave:v3` for slave, and the specified SHA for frontend.

2. **Service Selector Mismatch**:
   **Issue**: Services not routing to correct pods.
   **Fix**: Ensure selectors match deployment labels (`app`, `role`, `tier`).

3. **Environment Variable Errors**:
   **Issue**: Incorrect `GET_HOSTS_FROM` value.
   **Fix**: Set `GET_HOSTS_FROM` to `dns` for both `redis-slave` and `frontend`.

4. **Resource Requests**:
   **Issue**: Incorrect CPU or memory requests.
   **Fix**: Set `cpu: 100m` and `memory: 100Mi` for all containers.

5. **NodePort Conflict**:
   **Issue**: NodePort 30009 already in use.
   **Fix**: Ensure 30009 is within 30000-32767 and not conflicting.

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Configuration File

#### Guestbook Configuration (`guestbook-task5.yaml`)
Create or edit a file named `guestbook-task5.yaml`:

```yaml
# 1. Redis Master Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  namespace: default
  labels:
    app: guestbook
    tier: backend
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: guestbook
      role: master
  template:
    metadata:
      labels:
        app: guestbook
        role: master
    spec:
      containers:
      - name: master-redis-datacenter
        image: redis
        ports:
        - containerPort: 6379
          name: redis-port
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"

---
# 2. Redis Master Service
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  namespace: default
  labels:
    app: guestbook
    role: master
spec:
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
  selector:
    app: guestbook
    role: master

---
# 3. Redis Slave Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  namespace: default
  labels:
    app: guestbook
    tier: backend
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: guestbook
      role: slave
  template:
    metadata:
      labels:
        app: guestbook
        role: slave
    spec:
      containers:
      - name: slave-redis-datacenter
        image: gcr.io/google_samples/gb-redisslave:v3
        ports:
        - containerPort: 6379
          name: redis-port
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
        env:
        - name: GET_HOSTS_FROM
          value: "dns"

---
# 4. Redis Slave Service
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  namespace: default
  labels:
    app: guestbook
    role: slave
spec:
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
  selector:
    app: guestbook
    role: slave

---
# 5. Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis-datacenter
        image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
        ports:
        - containerPort: 80
          name: http-port
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
        env:
        - name: GET_HOSTS_FROM
          value: "dns"

---
# 6. Frontend Service
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: default
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30009
  selector:
    app: guestbook
    tier: frontend
```

**Configuration Breakdown**:
- **Redis Master Deployment**:
  - **Name**: `redis-master`
  - **Namespace**: `default`
  - **Labels**: `app: guestbook`, `tier: backend`, `role: master`
  - **Replicas**: 1
  - **Container**: `master-redis-datacenter`, image `redis`, port 6379
  - **Resources**: Requests 100m CPU, 100Mi memory
- **Redis Master Service**:
  - **Name**: `redis-master`
  - **Namespace**: `default`
  - **Selector**: `app: guestbook`, `role: master`
  - **Ports**: `port: 6379`, `targetPort: 6379`
- **Redis Slave Deployment**:
  - **Name**: `redis-slave`
  - **Namespace**: `default`
  - **Labels**: `app: guestbook`, `tier: backend`, `role: slave`
  - **Replicas**: 2
  - **Container**: `slave-redis-datacenter`, image `gcr.io/google_samples/gb-redisslave:v3`, port 6379
  - **Resources**: Requests 100m CPU, 100Mi memory
  - **Environment Variable**: `GET_HOSTS_FROM: dns`
- **Redis Slave Service**:
  - **Name**: `redis-slave`
  - **Namespace**: `default`
  - **Selector**: `app: guestbook`, `role: slave`
  - **Ports**: `port: 6379`, `targetPort: 6379`
- **Frontend Deployment**:
  - **Name**: `frontend`
  - **Namespace**: `default`
  - **Labels**: `app: guestbook`, `tier: frontend`
  - **Replicas**: 3
  - **Container**: `php-redis-datacenter`, image with specified SHA, port 80
  - **Resources**: Requests 100m CPU, 100Mi memory
  - **Environment Variable**: `GET_HOSTS_FROM: dns`
- **Frontend Service**:
  - **Name**: `frontend`
  - **Namespace**: `default`
  - **Type**: `NodePort`
  - **Selector**: `app: guestbook`, `tier: frontend`
  - **Ports**: `port: 80`, `targetPort: 80`, `nodePort: 30009`

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configurations
```bash
kubectl apply -f guestbook-task5.yaml
```

**Purpose**: Create the deployments and services in the Kubernetes cluster.

**Expected Output**:
```
deployment.apps/redis-master created
service/redis-master created
deployment.apps/redis-slave created
service/redis-slave created
deployment.apps/frontend created
service/frontend created
```

---

### Step 4: Verify Resources
```bash
kubectl get deployments.apps redis-master redis-slave frontend
```

**Expected Output**:
```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           2m
redis-slave    2/2     2            2           2m
frontend       3/3     3            3           2m
```

```bash
kubectl get pod -l app=guestbook
```

**Expected Output**:
```
NAME                            READY   STATUS    RESTARTS   AGE
redis-master-5d4b7c8f9-xyz123   1/1     Running   0          2m
redis-slave-6d9c8f7b4-abc456   1/1     Running   0          2m
redis-slave-6d9c8f7b4-def789   1/1     Running   0          2m
frontend-7b8d9e6c7-ghi012       1/1     Running   0          2m
frontend-7b8d9e6c7-jkl345       1/1     Running   0          2m
frontend-7b8d9e6c7-mno678       1/1     Running   0          2m
```

```bash
kubectl get svc redis-master redis-slave frontend
```

**Expected Output**:
```
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
redis-master   ClusterIP  10.96.123.456   <none>        6379/TCP         2m
redis-slave    ClusterIP  10.96.123.789   <none>        6379/TCP         2m
frontend       NodePort   10.96.123.012   <none>        80:30009/TCP     2m
```

```bash
kubectl describe svc frontend
```

**Expected Output**:
```
Name:                     frontend
Namespace:                default
Labels:                   app=guestbook
                          tier=frontend
Annotations:              <none>
Selector:                 app=guestbook,tier=frontend
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.012
IPs:                      10.96.123.012
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30009/TCP
Endpoints:                10.244.0.10:80,10.244.0.11:80,10.244.0.12:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

---

### Step 5: Verify Redis Master (optional)
```bash
POD=$(kubectl get pod -l app=guestbook,role=master -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping
```

**Expected Output**:
```
PONG
```

**Purpose**: Confirm the Redis master is operational.

---

### Step 6: Verify Redis Slave
```bash
POD=$(kubectl get pod -l app=guestbook,role=slave -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping
```

**Expected Output**:
```
PONG
```

**Purpose**: Confirm the Redis slave is operational.

---

### Step 7: Verify Application Accessibility
**Access the Application**:
- Click the **App** button in the lab interface and the following content will appear
![screenshot sample](Guestbook.png)


**Expected Output**: The guestbook application page is displayed.

**Purpose**: Confirm the guestbook application is accessible.

---

## 🔍 Code Analysis

### Guestbook Configuration (`guestbook-task5.yaml`)
```yaml
# 1. Redis Master Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  namespace: default
  labels:
    app: guestbook
    tier: backend
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: guestbook
      role: master
  template:
    metadata:
      labels:
        app: guestbook
        role: master
    spec:
      containers:
      - name: master-redis-datacenter
        image: redis
        ports:
        - containerPort: 6379
          name: redis-port
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"

---
# 2. Redis Master Service
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  namespace: default
  labels:
    app: guestbook
    role: master
spec:
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
  selector:
    app: guestbook
    role: master

---
# 3. Redis Slave Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  namespace: default
  labels:
    app: guestbook
    tier: backend
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: guestbook
      role: slave
  template:
    metadata:
      labels:
        app: guestbook
        role: slave
    spec:
      containers:
      - name: slave-redis-datacenter
        image: gcr.io/google_samples/gb-redisslave:v3
        ports:
        - containerPort: 6379
          name: redis-port
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
        env:
        - name: GET_HOSTS_FROM
          value: "dns"

---
# 4. Redis Slave Service
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  namespace: default
  labels:
    app: guestbook
    role: slave
spec:
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
  selector:
    app: guestbook
    role: slave

---
# 5. Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis-datacenter
        image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
        ports:
        - containerPort: 80
          name: http-port
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
        env:
        - name: GET_HOSTS_FROM
          value: "dns"

---
# 6. Frontend Service
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: default
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30009
  selector:
    app: guestbook
    tier: frontend
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **Redis Master Deployment** | redis-master | replicas: 1, app: guestbook, role: master | Runs Redis master |
| **Redis Master Container** | master-redis-datacenter | image: redis, port: 6379, cpu: 100m, memory: 100Mi | Redis master container |
| **Redis Master Service** | redis-master | port: 6379, selector: app=guestbook,role=master | Exposes Redis master |
| **Redis Slave Deployment** | redis-slave | replicas: 2, app: guestbook, role: slave | Runs Redis slaves |
| **Redis Slave Container** | slave-redis-datacenter | image: gcr.io/google_samples/gb-redisslave:v3, port: 6379, cpu: 100m, memory: 100Mi, GET_HOSTS_FROM: dns | Redis slave container |
| **Redis Slave Service** | redis-slave | port: 6379, selector: app=guestbook,role=slave | Exposes Redis slaves |
| **Frontend Deployment** | frontend | replicas: 3, app: guestbook, tier: frontend | Runs guestbook frontend |
| **Frontend Container** | php-redis-datacenter | image: gcr.io/google-samples/gb-frontend@sha256:..., port: 80, cpu: 100m, memory: 100Mi, GET_HOSTS_FROM: dns | Frontend container |
| **Frontend Service** | frontend | type: NodePort, port: 80, nodePort: 30009, selector: app=guestbook,tier=frontend | Exposes frontend externally |

---

## ✅ Verification Steps

### Step 1: Verify Deployments
```bash
kubectl get deployments.apps redis-master redis-slave frontend
```

**Expected Output**:
```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           2m
redis-slave    2/2     2            2           2m
frontend       3/3     3            3           2m
```

### Step 2: Verify Pods
```bash
kubectl get pod -l app=guestbook
```

**Expected Output**:
```
NAME                            READY   STATUS    RESTARTS   AGE
redis-master-5d4b7c8f9-xyz123   1/1     Running   0          2m
redis-slave-6d9c8f7b4-abc456   1/1     Running   0          2m
redis-slave-6d9c8f7b4-def789   1/1     Running   0          2m
frontend-7b8d9e6c7-ghi012       1/1     Running   0          2m
frontend-7b8d9e6c7-jkl345       1/1     Running   0          2m
frontend-7b8d9e6c7-mno678       1/1     Running   0          2m
```

### Step 3: Verify Services
```bash
kubectl get svc redis-master redis-slave frontend
```

**Expected Output**:
```
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
redis-master   ClusterIP  10.96.123.456   <none>        6379/TCP         2m
redis-slave    ClusterIP  10.96.123.789   <none>        6379/TCP         2m
frontend       NodePort   10.96.123.012   <none>        80:30009/TCP     2m
```

### Step 4: Verify Frontend Service Endpoints
```bash
kubectl describe svc frontend
```

**Expected Output**:
```
Name:                     frontend
Namespace:                default
Labels:                   app=guestbook
                          tier=frontend
Annotations:              <none>
Selector:                 app=guestbook,tier=frontend
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.012
IPs:                      10.96.123.012
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30009/TCP
Endpoints:                10.244.0.10:80,10.244.0.11:80,10.244.0.12:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Step 5: Verify Redis Master
```bash
POD=$(kubectl get pod -l app=guestbook,role=master -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping
```

**Expected Output**:
```
PONG
```

### Step 6: Verify Redis Slave
```bash
POD=$(kubectl get pod -l app=guestbook,role=slave -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping
```

**Expected Output**:
```
PONG
```

### Step 7: Verify Application
```bash
curl http://<node-ip>:30009
```

**Expected Output**: The guestbook application page is displayed.

---

## 🧪 Testing

### Verify Redis Master Port
```bash
POD=$(kubectl get pod -l app=guestbook,role=master -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN
```

**Purpose**: Confirm Redis master is listening on port 6379.

### Verify Redis Slave Port
```bash
POD=$(kubectl get pod -l app=guestbook,role=slave -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN
```

**Purpose**: Confirm Redis slave is listening on port 6379.

### Verify Frontend Port
```bash
POD=$(kubectl get pod -l app=guestbook,tier=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
```

**Purpose**: Confirm frontend is listening on port 80.

---

## 📚 Quick Command Reference
```bash
# Apply configurations
kubectl apply -f guestbook-task5.yaml

# Verify resources
kubectl get deployments.apps redis-master redis-slave frontend
kubectl get pod -l app=guestbook
kubectl get svc redis-master redis-slave frontend

# Verify Redis master
POD=$(kubectl get pod -l app=guestbook,role=master -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping

# Verify Redis slave
POD=$(kubectl get pod -l app=guestbook,role=slave -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- redis-cli ping

# Test application
curl http://<node-ip>:30009
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.  
**Solution**: Check pod events and logs.
```bash
kubectl describe pod -l app=guestbook
kubectl logs -l app=guestbook,role=master
kubectl logs -l app=guestbook,role=slave
kubectl logs -l app=guestbook,tier=frontend
```

### Issue 2: Service Has No Endpoints
**Symptoms**: `kubectl describe svc` shows `Endpoints: <none>`.  
**Solution**: Verify selector alignment.
```bash
kubectl get pod -l app=guestbook -o wide
kubectl describe svc redis-master
kubectl describe svc redis-slave
kubectl describe svc frontend
```

### Issue 3: Image Pull Failure
**Symptoms**: Pods fail with `ImagePullBackOff`.  
**Solution**: Verify image names and tags.
```bash
kubectl describe pod -l app=guestbook
```

### Issue 4: Application Not Accessible
**Symptoms**: `curl http://<node-ip>:30009` fails.  
**Solution**: Check frontend connectivity and Redis services.
```bash
POD=$(kubectl get pod -l app=guestbook,tier=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- curl localhost:80
kubectl describe svc redis-master redis-slave
```

---

## 💡 Additional Tips
- **Pod Logs**: Check for errors in Redis or frontend.
  ```bash
  kubectl logs -l app=guestbook,role=master
  kubectl logs -l app=guestbook,role=slave
  kubectl logs -l app=guestbook,tier=frontend
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f guestbook-task5.yaml --dry-run=client
  ```
- **NodePort Range**: Ensure 30009 is within 30000-32767.
- **Redis Health**: Verify Redis master and slave connectivity.
  ```bash
  POD=$(kubectl get pod -l app=guestbook,role=master -o jsonpath='{.items[0].metadata.name}')
  kubectl exec -it $POD -- redis-cli ping
  ```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Multi-Tier Architecture**: Configuring Redis master, slave, and frontend with proper connectivity.
2. **Environment Variables**: Setting `GET_HOSTS_FROM: dns` for Redis slave and frontend.
3. **Service Selectors**: Ensuring correct labels for service-to-pod mapping.
4. **NodePort Exposure**: Exposing the frontend on the specified NodePort.

### 💡 Solution Approach:
1. **Deployments**: Created `redis-master`, `redis-slave`, and `frontend` with specified replicas, images, and resources.
2. **Services**: Set up `redis-master`, `redis-slave` (ClusterIP), and `frontend` (NodePort 30009).
3. **Environment Variables**: Configured `GET_HOSTS_FROM: dns` for Redis slave and frontend.
4. **Verification**: Confirmed pod status, service endpoints, and application accessibility.

### 🎯 Key Success Factors:
- **Redis Master/Slave**: Properly configured with services for internal communication.
- **Frontend**: Scaled to 3 replicas and exposed via NodePort.
- **Application**: Accessible via the **App** button.
- **Resources**: Correctly set CPU and memory requests.

### ⚠️ Critical Configuration Details:
- **Redis Master**: `redis-master`, 1 replica, port 6379
- **Redis Slave**: `redis-slave`, 2 replicas, port 6379, `GET_HOSTS_FROM: dns`
- **Frontend**: `frontend`, 3 replicas, port 80, `GET_HOSTS_FROM: dns`, NodePort 30009
- **Resources**: 100m CPU, 100Mi memory for all containers

### 🔒 Kubernetes Benefits:
- **Scalability**: Multiple replicas for Redis slave and frontend.
- **Reliability**: Kubernetes manages pod health and restarts.
- **Service Discovery**: DNS-based service discovery via `GET_HOSTS_FROM`.
- **Accessibility**: NodePort enables external access.

---

## ⚠️ Important Production Notes
🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.  
🔐 **Security**: Implement network policies to restrict Redis and frontend access.  
📊 **Monitoring**: Monitor Redis replication and frontend performance.  
🛡️ **Persistence**: Add PersistentVolumes for Redis data in production.  
🌐 **Load Balancing**: Consider an Ingress or LoadBalancer for frontend traffic.

---

## 📖 Learning Outcomes
**Key Concepts Mastered**:
- **Deployments**: Managing multi-tier applications with multiple replicas.
- **Services**: Exposing backend and frontend via ClusterIP and NodePort.
- **Environment Variables**: Configuring service discovery with DNS.
- **Resource Management**: Setting CPU and memory requests.
- **Troubleshooting**: Verifying pod, service, and application health.

**Kubernetes Features Used**:
- `apps/v1 Deployment` for Redis and frontend.
- `v1 Service` for ClusterIP and NodePort exposure.

---

## 🎯 Task Completion Summary
✅ **Successfully Completed**:
- **Redis Master** - Created `redis-master` deployment and service.
- **Redis Slave** - Created `redis-slave` deployment (2 replicas) and service.
- **Frontend** - Created `frontend` deployment (3 replicas) and NodePort service (30009).
- **Verification** - Confirmed resources are running and guestbook application is accessible.

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Application Foundation**: The guestbook application is deployed with Redis master/slave backend and PHP frontend, accessible via NodePort 30009, ready for use by the Nautilus Application Development team.

### 🔮 Application Ready for:
- **Testing**: Validate guestbook functionality.
- **Scaling**: Adjust replicas for load handling.
- **Security**: Add network policies and authentication.
- **Monitoring**: Implement health checks and metrics.
- **Persistence**: Add storage for Redis data.