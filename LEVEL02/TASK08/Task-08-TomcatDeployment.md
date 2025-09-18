**🌟 Task 8 - Deploy Tomcat with NodePort Service**

**📌 Task Description**

The Nautilus DevOps team is deploying a Java-based Tomcat application on a Kubernetes cluster.

**Requirements:**
- Create namespace `tomcat-namespace-xfusion`.
- Create deployment `tomcat-deployment-xfusion` in `tomcat-namespace-xfusion`, container `tomcat-container-xfusion`, image `gcr.io/kodekloud/centos-ssh-enabled:tomcat`, port `8080`, 1 replica.
- Create `NodePort` service `tomcat-service-xfusion` in `tomcat-namespace-xfusion`, nodePort `32227`.
- Ensure application is running and accessible via the "App" button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like names or ports may differ in your environment.

👉 **Your task**: Deploy Tomcat and expose it via a NodePort service.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Namespace

```bash
kubectl create namespace tomcat-namespace-xfusion
kubectl config set-context --current --namespace=tomcat-namespace-xfusion
```

**Purpose**: Create and set the `tomcat-namespace-xfusion` namespace.

**Expected Output**:
```
namespace/tomcat-namespace-xfusion created
Context "current-context" modified.
```

---

## 🔹 Step 3: Create Deployment and Service Manifest

```bash
vi tomcat-deployment.yaml
```

**Purpose**: Define the Tomcat deployment and service.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment-xfusion
  namespace: tomcat-namespace-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat-app
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      containers:
      - name: tomcat-container-xfusion
        image: gcr.io/kodekloud/centos-ssh-enabled:tomcat
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service-xfusion
  namespace: tomcat-namespace-xfusion
spec:
  type: NodePort
  selector:
    app: tomcat-app
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
    nodePort: 32227
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Deployment`: Defines Tomcat deployment with 1 replica.
- `labels` and `selector`: Uses `app: tomcat-app` for matching.
- `kind: Service`: Exposes deployment on nodePort 32227.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 4: Apply Manifests

```bash
kubectl apply -f tomcat-deployment.yaml
```

**Purpose**: Deploy the resources.

**Expected Output**:
```
deployment.apps/tomcat-deployment-xfusion created
service/tomcat-service-xfusion created
```

---

## 🔹 Step 5: Verify Deployment

```bash
kubectl get deployment
```

**Purpose**: Confirm deployment is ready.

**Expected Output**:
```
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
tomcat-deployment-xfusion   1/1     1            1           10s
```

---

## 🔹 Step 6: Verify Pods

```bash
kubectl get pods
```

**Purpose**: Ensure pod is running.

**Expected Output**:
```
NAME                                         READY   STATUS    RESTARTS   AGE
tomcat-deployment-xfusion-xxx-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 7: Verify Service

```bash
kubectl get svc
```

**Purpose**: Confirm service configuration.

**Expected Output**:
```
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
tomcat-service-xfusion   NodePort   10.96.x.x       <none>        8080:32227/TCP   10s
```

---

## 🔹 Step 8: Verify Tomcat Accessibility

```bash
curl http://<node-ip>:32227
```

**Purpose**: Test the Tomcat application.

**Expected Output**:
```
Welcome to xFusionCorp Industries!
```

**Note**: Click the "App" button to verify in the lab environment.

---

## 📋 Quick Command Reference

```bash
# Create namespace and apply manifest
kubectl create namespace tomcat-namespace-xfusion
kubectl config set-context --current --namespace=tomcat-namespace-xfusion
vi tomcat-deployment.yaml
# (Paste YAML content)
kubectl apply -f tomcat-deployment.yaml

# Verify resources
kubectl get deployment
kubectl get pods
kubectl get svc
curl http://<node-ip>:32227
```

---

## 💡 Additional Tips

- **Task Variability**: Namespace or nodePort may differ; adjust YAML as needed.
- **Tomcat Startup**: May take time to initialize; monitor pod status.
- **Access Control**: Secure Tomcat with authentication in production.
- **Dry Run**: Test with `kubectl apply -f tomcat-deployment.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod tomcat-deployment-xfusion-xxx-xxx
kubectl logs tomcat-deployment-xfusion-xxx-xxx
```

### **Issue 2: Service Inaccessible**
**Symptoms**: `curl` fails on nodePort 32227.
**Solution**: Verify service configuration.
```bash
kubectl describe svc tomcat-service-xfusion
# Ensure nodePort: 32227 and selector: app=tomcat-app
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod tomcat-deployment-xfusion-xxx-xxx
# Ensure image: gcr.io/kodekloud/centos-ssh-enabled:tomcat
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying a Tomcat application** and ensuring accessibility via a NodePort service.

**💡 Solution Approach:**
1. **Namespace Setup**: Created `tomcat-namespace-xfusion`.
2. **Deployment Configuration**: Used specified Tomcat image with 1 replica.
3. **Service Exposure**: Configured nodePort 32227.
4. **Verification**: Confirmed pod status and application accessibility.

**🎯 Key Success Factors:**
- **Correct image**: Used `gcr.io/kodekloud/centos-ssh-enabled:tomcat`.
- **Service access**: NodePort 32227 exposes Tomcat.
- **Pod status**: Ensured `Running` state.
- **Namespace isolation**: Used `tomcat-namespace-xfusion`.

**⚠️ Critical Configuration Details:**
- **Namespace**: Must be `tomcat-namespace-xfusion`.
- **Deployment Name**: Must be `tomcat-deployment-xfusion`.
- **Container Name**: Must be `tomcat-container-xfusion`.
- **Image**: Must be `gcr.io/kodekloud/centos-ssh-enabled:tomcat`.
- **Service**: `tomcat-service-xfusion` with nodePort 32227.
- **Port**: Container port 8080.

**🔒 Kubernetes Benefits:**
- **Application Hosting**: Supports Java-based apps like Tomcat.
- **Exposability**: NodePort enables external access.
- **Isolation**: Namespaces separate workloads.

---

## ⚠️ Important Production Notes

🔧 **Tomcat Configuration**: Secure with authentication and persistent storage.

🔐 **Security**: Use Secrets for Tomcat credentials.

📊 **Monitoring**: Monitor Tomcat pod health and resource usage.

🛡️ **Scalability**: Consider scaling for high-traffic applications.

---
