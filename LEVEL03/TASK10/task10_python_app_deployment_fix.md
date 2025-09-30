# 🌟 Task 10 - Fix Python Application Deployment on Kubernetes Cluster

## 📌 Task Description

One of the Nautilus DevOps team members attempted to deploy a Python application on a Kubernetes cluster, but due to misconfiguration, the application is not coming up. The task is to fix the issues in the deployment and service configurations to ensure the application is accessible on the specified NodePort.

### Requirements:
- **Deployment**: Fix `python-deployment-datacenter` using the image `poroko/flask-demo-app`.
- **Service**: Fix `python-service-datacenter` to use:
  - `nodePort: 32345`
  - `targetPort`: Default Flask port (5000)
- **Verification**: Ensure the application is accessible via the **App** button and displays "Hello World Pyvo 1!".
- **Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. The deployment and service are already deployed but need to be fixed.

👉 **Your task**: Fix the misconfigurations in the `python-deployment-datacenter` deployment and `python-service-datacenter` service, apply the changes, and verify that the application is accessible.

---

## 🔧 Infrastructure Overview

**Target Environment**: Kubernetes Cluster
**Resources**:
- Deployment: `python-deployment-datacenter` for the Python Flask application
- Service: `python-service-datacenter` to expose the application
- Namespace: Default

**Working Directory**: Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Deployment**: Runs the Python Flask application
- **Service**: Exposes the application on NodePort 32345
- **Application**: Flask-based Python app accessible via HTTP

### 🎯 Implementation Strategy
1. Identify and fix errors in the deployment and service configurations
2. Update the deployment to use the correct image (`poroko/flask-demo-app`)
3. Update the service to use the correct ports (`port: 5000`, `targetPort: 5000`, `nodePort: 32345`)
4. Apply the corrected configurations
5. Verify resource creation and application accessibility

---

## 🚨 Errors in the Provided Solution

The provided configurations contain the following errors:

1. **Image Mismatch in Deployment**:
   ```yaml
   image: poroko/flask-app-demo
   ```
   **Issue**: The deployment uses the incorrect image `poroko/flask-app-demo`, while the task requires `poroko/flask-demo-app`.
   **Fix**: Update the image to `poroko/flask-demo-app`.

2. **Service Port Mismatch**:
   ```yaml
   ports:
     - port: 8080
       targetPort: 8080
       nodePort: 32345
   ```
   **Issue**: The service is configured with `port: 8080` and `targetPort: 8080`, but the Flask application listens on port 5000 (as indicated by `containerPort: 5000` in the deployment). This causes no endpoints to be associated with the service.
   **Fix**: Update the service to use `port: 5000` and `targetPort: 5000`, keeping `nodePort: 32345`.

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

### Step 2: Verify Current State
Check the current deployment and service status:
```bash
kubectl get deployments.apps python-deployment-datacenter
kubectl describe deployments.apps python-deployment-datacenter
kubectl get svc python-service-datacenter
kubectl describe svc python-service-datacenter
```

**Purpose**: Confirm the misconfigurations (incorrect image and port mismatch).

---

### Step 3: Create Corrected Configuration Files

#### Python Deployment (`python-deployment-datacenter.yaml`)
Create or edit a file named `python-deployment-datacenter.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-deployment-datacenter
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python_app
  template:
    metadata:
      labels:
        app: python_app
    spec:
      containers:
        - name: python-container-datacenter
          image: poroko/flask-demo-app
          ports:
            - containerPort: 5000
              name: http
```

**Configuration Breakdown**:
- **Deployment**: `python-deployment-datacenter`
- **Namespace**: `default`
- **Labels**: `app: python_app`
- **Replicas**: 1
- **Selector**: `matchLabels` with `app: python_app`
- **Container**: `python-container-datacenter` with image `poroko/flask-demo-app`
- **Ports**: Exposes port 5000 for Flask

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

#### Python Service (`python-service-datacenter.yaml`)
Create or edit a file named `python-service-datacenter.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-datacenter
  namespace: default
spec:
  selector:
    app: python_app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 32345
  type: NodePort
```

**Configuration Breakdown**:
- **Service**: `python-service-datacenter`
- **Namespace**: `default`
- **Selector**: `app: python_app`
- **Ports**: `port: 5000`, `targetPort: 5000`, `nodePort: 32345`
- **Type**: NodePort

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 4: Apply Configurations
```bash
kubectl apply -f python-deployment-datacenter.yaml
kubectl apply -f python-service-datacenter.yaml
```

**Purpose**: Update the deployment with the correct image and the service with the correct ports.

**Expected Output**:
```
deployment.apps/python-deployment-datacenter configured
service/python-service-datacenter configured
```

---

### Step 5: Verify Resources
```bash
kubectl get deployments.apps python-deployment-datacenter
```

**Expected Output**:
```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
python-deployment-datacenter   1/1     1            1           16m
```

```bash
kubectl get pod -l app=python_app
```

**Expected Output**:
```
NAME                                           READY   STATUS    RESTARTS   AGE
python-deployment-datacenter-6fdb496d59-xyz123 1/1     Running   0          16m
```

```bash
kubectl get svc python-service-datacenter
```

**Expected Output**:
```
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
python-service-datacenter   NodePort    10.96.136.196   <none>        5000:32345/TCP   16m
```

```bash
kubectl describe svc python-service-datacenter
```

**Expected Output**:
```
Name:                     python-service-datacenter
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=python_app
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.136.196
IPs:                      10.96.136.196
Port:                     <unset>  5000/TCP
TargetPort:               5000/TCP
NodePort:                 <unset>  32345/TCP
Endpoints:                10.244.0.6:5000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

---

### Step 6: Verify Application Accessibility
**Access the Application**:
- Click the **App** button in the lab interface.
- Alternatively, test with:
```bash
curl http://<node-ip>:32345
```

**Expected Output**:
```
Hello World Pyvo 1!
```

**Verification**: Confirm the application displays "Hello World Pyvo 1!".

---

## 🔍 Code Analysis

### Python Deployment (`python-deployment-datacenter.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-deployment-datacenter
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python_app
  template:
    metadata:
      labels:
        app: python_app
    spec:
      containers:
        - name: python-container-datacenter
          image: poroko/flask-demo-app
          ports:
            - containerPort: 5000
              name: http
```

### Python Service (`python-service-datacenter.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-datacenter
  namespace: default
spec:
  selector:
    app: python_app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 32345
  type: NodePort
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **Deployment** | python-deployment-datacenter | replicas: 1, app: python_app | Runs Flask application |
| **Container** | python-container-datacenter | image: poroko/flask-demo-app | Flask application container |
| **Container Port** | http | 5000 | Default Flask port |
| **Service** | python-service-datacenter | type: NodePort | Exposes application externally |
| **Service Ports** | port, targetPort, nodePort | 5000, 5000, 32345 | Routes traffic to Flask app |

---

## ✅ Verification Steps

### Step 1: Verify Deployment
```bash
kubectl get deployments.apps python-deployment-datacenter
```

**Expected Output**:
```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
python-deployment-datacenter   1/1     1            1           16m
```

### Step 2: Verify Pods
```bash
kubectl get pod -l app=python_app
```

**Expected Output**:
```
NAME                                           READY   STATUS    RESTARTS   AGE
python-deployment-datacenter-6fdb496d59-xyz123 1/1     Running   0          16m
```

### Step 3: Verify Service
```bash
kubectl get svc python-service-datacenter
```

**Expected Output**:
```
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
python-service-datacenter   NodePort    10.96.136.196   <none>        5000:32345/TCP   16m
```

### Step 4: Verify Service Endpoints
```bash
kubectl describe svc python-service-datacenter
```

**Expected Output**:
```
Name:                     python-service-datacenter
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=python_app
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.136.196
IPs:                      10.96.136.196
Port:                     <unset>  5000/TCP
TargetPort:               5000/TCP
NodePort:                 <unset>  32345/TCP
Endpoints:                10.244.0.6:5000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Step 5: Verify Application
```bash
curl http://<node-ip>:32345
```

**Expected Output**:
```
Hello World Pyvo 1!
```

---

## 🧪 Testing

### Verify Container Port
```bash
POD=$(kubectl get pod -l app=python_app -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- netstat -tuln
```

**Purpose**: Confirm the Flask app is listening on port 5000.

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN
```

### Verify Service Configuration
```bash
kubectl describe svc python-service-datacenter
```

**Purpose**: Confirm the service has the correct ports and endpoints.

---

## 📚 Quick Command Reference
```bash
# Verify current state
kubectl get deployments.apps python-deployment-datacenter
kubectl describe deployments.apps python-deployment-datacenter
kubectl get svc python-service-datacenter
kubectl describe svc python-service-datacenter

# Apply configurations
kubectl apply -f python-deployment-datacenter.yaml
kubectl apply -f python-service-datacenter.yaml

# Verify resources
kubectl get deployments.apps python-deployment-datacenter
kubectl get pod -l app=python_app
kubectl get svc python-service-datacenter

# Test application
curl http://<node-ip>:32345
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pods Not Running
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod -l app=python_app
kubectl logs -l app=python_app
```

### Issue 2: Service Has No Endpoints
**Symptoms**: `kubectl describe svc python-service-datacenter` shows `Endpoints: <none>`.
**Solution**: Verify selector and port alignment.
```bash
kubectl get pod -l app=python_app -o wide
kubectl describe svc python-service-datacenter
```

### Issue 3: Incorrect NodePort
**Symptoms**: Service uses the wrong port.
**Solution**: Patch the service.
```bash
kubectl patch svc python-service-datacenter -p '{"spec":{"ports":[{"port":5000,"targetPort":5000,"nodePort":32345}]}}'
```

### Issue 4: Image Pull Failure
**Symptoms**: Pod fails with `ImagePullBackOff`.
**Solution**: Verify the image name.
```bash
kubectl describe pod -l app=python_app
```

---

## 💡 Additional Tips
- **Pod Logs**: Check for runtime issues.
  ```bash
  kubectl logs -l app=python_app
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f python-deployment-datacenter.yaml --dry-run=client
  kubectl apply -f python-service-datacenter.yaml --dry-run=client
  ```
- **NodePort Range**: Ensure 32345 is within 30000-32767.
- **Application Health**: Verify Flask is running.
  ```bash
  POD=$(kubectl get pod -l app=python_app -o jsonpath='{.items[0].metadata.name}')
  kubectl exec $POD -- curl localhost:5000
  ```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Image Mismatch**: Incorrect image name in the deployment.
2. **Port Mismatch**: Service ports did not match the Flask application’s port.
3. **Service Connectivity**: No endpoints due to port mismatch.

### 💡 Solution Approach:
1. **Image Fix**: Updated the deployment to use `poroko/flask-demo-app`.
2. **Port Fix**: Updated the service to use `port: 5000` and `targetPort: 5000`.
3. **Verification**: Confirmed deployment, service, and application accessibility.

### 🎯 Key Success Factors:
- **Deployment**: Correct image ensures the Flask app runs.
- **Service**: Correct ports ensure traffic reaches the app.
- **Application**: Displays "Hello World Pyvo 1!".

### ⚠️ Critical Configuration Details:
- **Deployment**: `python-deployment-datacenter`
- **Service**: `python-service-datacenter`
- **Image**: `poroko/flask-demo-app`
- **Ports**: `port: 5000`, `targetPort: 5000`, `nodePort: 32345`

### 🔒 Kubernetes Benefits:
- **Scalability**: Deployment supports replicas
- **Accessibility**: NodePort enables external access
- **Reliability**: Kubernetes manages pod health

---

## ⚠️ Important Production Notes
🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.
🔐 **Security**: Implement network policies for application access.
📊 **Monitoring**: Monitor Flask application health.
🛡️ **Scalability**: Consider multiple replicas for high availability.

---

## 📖 Learning Outcomes
**Key Concepts Mastered**:
- **Deployment**: Managing Flask application pods
- **Service**: Exposing applications via NodePort
- **Troubleshooting**: Fixing image and port misconfigurations
- **Verification**: Ensuring application accessibility

**Kubernetes Features Used**:
- `apps/v1 Deployment` for Flask application
- `v1 Service` for NodePort exposure

---

## 🎯 Task Completion Summary
✅ **Successfully Completed**:
- **Deployment** - Fixed `python-deployment-datacenter` with correct image
- **Service** - Fixed `python-service-datacenter` with correct ports
- **Verification** - Confirmed application displays "Hello World Pyvo 1!"

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Application Foundation**: The Python Flask application is deployed, accessible via NodePort 32345, and displays "Hello World Pyvo 1!", ready for further enhancements by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Scaling**: Increase replicas or use autoscaling
- **Security**: Add network policies
- **Monitoring**: Implement health checks and logging
- **Enhancements**: Add additional application features 