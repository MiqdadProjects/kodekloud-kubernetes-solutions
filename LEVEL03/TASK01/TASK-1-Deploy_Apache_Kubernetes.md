# 🌟 Task 1 - Deploy Apache Web Server on Kubernetes Cluster

## 📌 Task Description

The Nautilus application development team has requested the DevOps team to deploy an application on a Kubernetes cluster using the Apache web server. To meet their requirements, a Kubernetes deployment and service need to be created to ensure the application is accessible and scalable.

### Requirements:
- Create a namespace named **`httpd-namespace-nautilus`**.
- Create a deployment named **`httpd-deployment-nautilus`** under the newly created namespace, using the **`httpd:latest`** image with 2 replicas.
- Create a service named **`httpd-service-nautilus`** under the same namespace to expose the deployment on **NodePort 30004**.
- Ensure the `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Deploy an Apache web server application on the Kubernetes cluster using the specified namespace, deployment, and service configurations.

💡 **Note**: The deployment must use the `httpd:latest` image, and the service must expose the application externally on NodePort `30004`.

---

## 🔧 Infrastructure Overview

**Target Environment:** Kubernetes Cluster
**Resources:**
- Namespace for resource isolation
- Deployment for running Apache web server with 2 replicas
- Service to expose the Apache deployment on NodePort 30004

**Working Directory:** Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **Namespace**: Isolates resources for the Apache application
- **Deployment**: Manages Apache web server pods with `httpd:latest` image
- **Service**: Exposes the deployment externally via NodePort
- **Scalability**: Ensures 2 replicas for high availability

### 🎯 Implementation Strategy
1. Create the `httpd-namespace-nautilus` namespace
2. Create a deployment with 2 replicas using the `httpd:latest` image
3. Expose the deployment via a NodePort service on port 30004
4. Verify the deployment, pods, and service functionality
5. Test application accessibility

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose:** Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Namespace
```bash
kubectl create namespace httpd-namespace-nautilus
```

**Purpose:** Create the namespace to isolate resources.

**Expected Output:**
```
namespace/httpd-namespace-nautilus created
```

**Set Namespace Context**:
```bash
kubectl config set-context --current --namespace=httpd-namespace-nautilus
```

**Purpose:** Set the current context to the new namespace for easier management.

**Expected Output:**
```
Context "current-context" modified.
```

---

### Step 3: Create Deployment and Service Configuration
Create a file named `httpd-deployment.yaml` with the following content:

```yaml
# httpd-deployment.yaml

# Create deployment for Apache web server
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment-nautilus
  namespace: httpd-namespace-nautilus
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd-app
  template:
    metadata:
      labels:
        app: httpd-app
    spec:
      containers:
        - name: httpd-container
          image: httpd:latest
          ports:
            - containerPort: 80
---
# Create service to expose the deployment
apiVersion: v1
kind: Service
metadata:
  name: httpd-service-nautilus
  namespace: httpd-namespace-nautilus
spec:
  type: NodePort
  selector:
    app: httpd-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30004
```

**Configuration Breakdown:**
- **Deployment**:
  - `name: httpd-deployment-nautilus` - Specifies the deployment name
  - `namespace: httpd-namespace-nautilus` - Places the deployment in the specified namespace
  - `replicas: 2` - Ensures 2 pods are running
  - `image: httpd:latest` - Uses the specified Apache image
  - `selector` and `labels` - Matches pods to the deployment using the `httpd-app` label
- **Service**:
  - `name: httpd-service-nautilus` - Specifies the service name
  - `type: NodePort` - Exposes the service externally
  - `nodePort: 30004` - Sets the external port as required
  - `selector: app: httpd-app` - Links the service to the deployment's pods

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 4: Apply Configuration
```bash
kubectl apply -f httpd-deployment.yaml
```

**Purpose:** Deploy the namespace, deployment, and service to the Kubernetes cluster.

**Expected Output:**
```
deployment.apps/httpd-deployment-nautilus created
service/httpd-service-nautilus created
```

---

### Step 5: Verify Deployment and Pods
```bash
kubectl get deployment
```

**Purpose:** Check the status of the `httpd-deployment-nautilus` deployment.

**Expected Output:**
```
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
httpd-deployment-nautilus   2/2     2            2           1m
```

**Verification:** Confirm the deployment shows `2/2` ready replicas.

```bash
kubectl get pods
```

**Purpose:** Ensure pods are in a `Running` state.

**Expected Output:**
```
NAME                                        READY   STATUS    RESTARTS   AGE
httpd-deployment-nautilus-7b9f8d4c5-abcde   1/1     Running   0          1m
httpd-deployment-nautilus-7b9f8d4c5-xyz12   1/1     Running   0          1m
```

**Verification:** Confirm both pods are in `Running` state.

---

### Step 6: Verify Service Configuration
```bash
kubectl get svc httpd-service-nautilus
```

**Purpose:** Check the NodePort configuration of `httpd-service-nautilus`.

**Expected Output:**
```
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
httpd-service-nautilus   NodePort   10.96.123.456   <none>        80:30004/TCP   1m
```

**Verification:** Confirm the service uses NodePort `30004`.

---

### Step 7: Verify Application Accessibility
**Access the Application**:
- Click on the **Website** button provided in the lab interface.
- Alternatively, use the cluster's external IP or Node IP with port `30004` (e.g., via a browser or `curl`).
- Example: `curl http://<node-ip>:30004`

**Expected Output:**
```
It works!
```

**Verification:** Confirm the application is accessible and displays the expected Apache default page.

---

## 🔍 Code Analysis

### Complete Configuration (`httpd-deployment.yaml`)
```yaml
# Create deployment for Apache web server
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment-nautilus
  namespace: httpd-namespace-nautilus
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd-app
  template:
    metadata:
      labels:
        app: httpd-app
    spec:
      containers:
        - name: httpd-container
          image: httpd:latest
          ports:
            - containerPort: 80
---
# Create service to expose the deployment
apiVersion: v1
kind: Service
metadata:
  name: httpd-service-nautilus
  namespace: httpd-namespace-nautilus
spec:
  type: NodePort
  selector:
    app: httpd-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30004
```

### Deployment Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | httpd-deployment-nautilus | Name of the deployment |
| **namespace** | httpd-namespace-nautilus | Namespace for resource isolation |
| **replicas** | 2 | Number of pod replicas |
| **image** | httpd:latest | Apache web server image |
| **selector** | app: httpd-app | Matches pods to the deployment |

### Service Attributes
| Attribute | Value | Description |
|-----------|-------|-------------|
| **name** | httpd-service-nautilus | Name of the service |
| **namespace** | httpd-namespace-nautilus | Namespace for the service |
| **type** | NodePort | Exposes service externally |
| **nodePort** | 30004 | External port for accessing the application |
| **selector** | app: httpd-app | Links service to deployment pods |

### Default Properties
- **Deployment**: Runs 2 replicas with `httpd:latest` image
- **Service**: Exposes port 80 internally, mapped to NodePort 30004
- **Labels**: Uses `app: httpd-app` for pod selection
- **Container Port**: Apache listens on port 80

---

## ✅ Verification Steps

### Step 1: Verify Namespace Creation
```bash
kubectl get namespace httpd-namespace-nautilus
```

**Expected Output:**
```
NAME                     STATUS   AGE
httpd-namespace-nautilus Active   2m
```

### Step 2: Verify Deployment
```bash
kubectl get deployment httpd-deployment-nautilus
```

**Expected Output:**
```
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
httpd-deployment-nautilus   2/2     2            2           2m
```

### Step 3: Verify Pods
```bash
kubectl get pods -n httpd-namespace-nautilus
```

**Expected Output:**
```
NAME                                        READY   STATUS    RESTARTS   AGE
httpd-deployment-nautilus-7b9f8d4c5-abcde   1/1     Running   0          2m
httpd-deployment-nautilus-7b9f8d4c5-xyz12   1/1     Running   0          2m
```

### Step 4: Verify Service
```bash
kubectl get svc httpd-service-nautilus -n httpd-namespace-nautilus
```

**Expected Output:**
```
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
httpd-service-nautilus   NodePort   10.96.123.456   <none>        80:30004/TCP   2m
```

### Step 5: Test Application
```bash
curl http://<node-ip>:30004
```

**Expected Output:**
```
It works!
```

---

## 🧪 Testing

### Verify Deployment Status
```bash
kubectl describe deployment httpd-deployment-nautilus -n httpd-namespace-nautilus
```

**Purpose:** Check detailed deployment status and events.

### Verify Pod Status
```bash
kubectl describe pod -l app=httpd-app -n httpd-namespace-nautilus
```

**Purpose:** Confirm pods are running and check for any issues.

### Test Service Accessibility
```bash
kubectl get nodes -o wide
curl http://<node-ip>:30004
```

**Purpose:** Use the node IP to access the application and confirm output.

---

## 📚 Quick Command Reference

```bash
# Create namespace
kubectl create namespace httpd-namespace-nautilus
kubectl config set-context --current --namespace=httpd-namespace-nautilus

# Apply configuration
kubectl apply -f httpd-deployment.yaml

# Verify deployment, pods, and service
kubectl get deployment -n httpd-namespace-nautilus
kubectl get pods -n httpd-namespace-nautilus
kubectl get svc httpd-service-nautilus -n httpd-namespace-nautilus

# Test application
curl http://<node-ip>:30004
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Namespace Not Found
**Symptoms:** Error: "namespace not found"
**Solution:** Ensure namespace is created.
```bash
kubectl create namespace httpd-namespace-nautilus
```

### Issue 2: Pods Not Running
**Symptoms:** Pods in `Pending` or `CrashLoopBackOff`.
**Solution:** Check pod events and logs.
```bash
kubectl describe pod -n httpd-namespace-nautilus
kubectl logs -l app=httpd-app -n httpd-namespace-nautilus
```

### Issue 3: Service Not Accessible
**Symptoms:** Unable to access application on NodePort 30004.
**Solution:** Verify service configuration and node IP.
```bash
kubectl describe svc httpd-service-nautilus -n httpd-namespace-nautilus
kubectl get nodes -o wide
```

### Issue 4: Incorrect NodePort
**Symptoms:** Service uses wrong NodePort.
**Solution:** Patch the service to use 30004.
```bash
kubectl patch svc httpd-service-nautilus -n httpd-namespace-nautilus -p '{"spec": {"ports": [{"port":80,"targetPort":80,"nodePort":30004}]}}'
```

---

## 💡 Additional Tips

- **Pod Logs**: Check logs for runtime issues.
  ```bash
  kubectl logs -l app=httpd-app -n httpd-namespace-nautilus
  ```
- **Dry Run**: Test configuration changes before applying.
  ```bash
  kubectl apply -f httpd-deployment.yaml --dry-run=client
  ```
- **Namespace Context**: Ensure commands are executed in the correct namespace.
  ```bash
  kubectl config set-context --current --namespace=httpd-namespace-nautilus
  ```
- **NodePort Range**: Verify NodePort 30004 is within the cluster’s allowed range (30000-32767).

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Namespace Isolation**: Ensuring all resources are created in the `httpd-namespace-nautilus` namespace.
2. **Correct Image and Replicas**: Using `httpd:latest` and maintaining 2 replicas.
3. **Service Exposure**: Configuring NodePort 30004 correctly.

### 💡 Solution Approach:
1. **Created Namespace**: Isolated resources using `httpd-namespace-nautilus`.
2. **Defined Deployment**: Used `httpd:latest` with 2 replicas and proper labels.
3. **Configured Service**: Set up NodePort service to expose the application on port 30004.
4. **Verified Accessibility**: Ensured the application displays "It works!" via NodePort.

### 🎯 Key Success Factors:
- **Namespace**: Resources isolated in `httpd-namespace-nautilus`
- **Deployment**: 2 replicas running `httpd:latest`
- **Service**: Exposed on NodePort 30004
- **Application Output**: Displays "It works!"

### ⚠️ Critical Configuration Details:
- **Namespace**: Must be `httpd-namespace-nautilus`
- **Deployment Name**: Must be `httpd-deployment-nautilus`
- **Service Name**: Must be `httpd-service-nautilus`
- **NodePort**: Must be `30004`
- **Image**: Must be `httpd:latest`

### 🔒 Kubernetes Benefits:
- **Scalability**: 2 replicas ensure high availability
- **Accessibility**: NodePort service enables external access
- **Isolation**: Namespace provides resource isolation
- **Reliability**: Kubernetes manages pod health and restarts

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use `RollingUpdate` for zero-downtime updates.

🔐 **Security**: Implement network policies to restrict access to the service.

📊 **Monitoring**: Monitor pod health and HTTP request metrics.

🛡️ **Scalability**: Adjust replicas based on load or use Horizontal Pod Autoscaler.

---

## 📖 Learning Outcomes

**Key Concepts Mastered:**
- **Kubernetes Namespace**: Resource isolation for better organization
- **Deployment**: Managing scalable application pods
- **Service**: Exposing applications externally via NodePort
- **Apache Web Server**: Deploying a simple web server in Kubernetes

**Kubernetes Features Used:**
- `apps/v1 Deployment` - For managing Apache pods
- `v1 Service` - For exposing the application
- Namespace for resource isolation
- `kubectl` for cluster management

---

## 🎯 Task Completion Summary

✅ **Successfully Completed:**
- **Namespace** - Created `httpd-namespace-nautilus`
- **Deployment** - Created `httpd-deployment-nautilus` with 2 replicas using `httpd:latest`
- **Service** - Created `httpd-service-nautilus` with NodePort 30004
- **Verification** - Confirmed deployment, pods, and service functionality
- **Application Accessibility** - Verified output "It works!" via NodePort 30004

**Final Status:** 🎉 **Task completed successfully with all requirements met**

**Application Foundation:** The Apache web server is now deployed in the Kubernetes cluster under the `httpd-namespace-nautilus` namespace, running 2 replicas and accessible via NodePort 30004, ready for further configuration or scaling as needed by the Nautilus application development team.

### 🔮 Application Ready for:
- **Scaling**: Increase replicas or implement autoscaling
- **Monitoring**: Add health checks and metrics
- **Security**: Apply network policies and access controls
- **Content Updates**: Customize Apache content in `/usr/local/apache2/htdocs`