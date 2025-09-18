**🌟 Task 6 - Deploy Jenkins with NodePort Service**

**📌 Task Description**

The Nautilus DevOps team is setting up a Jenkins CI server on a Kubernetes cluster to manage deployment pipelines.

**Requirements:**
- Create namespace `jenkins`.
- Create `NodePort` service `jenkins-service` in `jenkins` namespace, nodePort `30008`.
- Create deployment `jenkins-deployment` in `jenkins` namespace, label `app: jenkins`, container `jenkins-container`, image `jenkins/jenkins`, port `8080`, 1 replica.
- Ensure pods are running and Jenkins login page is accessible via the "Jenkins" button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like names or ports may differ in your environment.

👉 **Your task**: Deploy Jenkins and expose it via a NodePort service.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Namespace

```bash
kubectl create namespace jenkins
kubectl config set-context --current --namespace=jenkins
```

**Purpose**: Create and set the `jenkins` namespace.

**Expected Output**:
```
namespace/jenkins created
Context "current-context" modified.
```

---

## 🔹 Step 3: Create Deployment Manifest

```bash
vi jenkins-deployment.yaml
```

**Purpose**: Define the Jenkins deployment.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  namespace: jenkins
  labels:
    app: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins-container
        image: jenkins/jenkins
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 4: Create Service Manifest

```bash
vi jenkins-service.yaml
```

**Purpose**: Define the NodePort service.

**YAML Content**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: jenkins
spec:
  type: NodePort
  selector:
    app: jenkins
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
    nodePort: 30008
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Deployment`: Defines Jenkins deployment with 1 replica.
- `labels` and `selector`: Uses `app: jenkins` for matching.
- `kind: Service`: Exposes deployment on nodePort 30008.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 5: Apply Manifests

```bash
kubectl apply -f jenkins-deployment.yaml
kubectl apply -f jenkins-service.yaml
```

**Purpose**: Deploy the resources.

**Expected Output**:
```
deployment.apps/jenkins-deployment created
service/jenkins-service created
```

---

## 🔹 Step 6: Verify Deployment

```bash
kubectl get deployment
```

**Purpose**: Confirm deployment is ready.

**Expected Output**:
```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
jenkins-deployment  1/1     1            1           10s
```

---

## 🔹 Step 7: Verify Pods

```bash
kubectl get pods
```

**Purpose**: Ensure pod is running.

**Expected Output**:
```
NAME                                 READY   STATUS    RESTARTS   AGE
jenkins-deployment-xxx-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 8: Verify Service

```bash
kubectl get svc
```

**Purpose**: Confirm service configuration.

**Expected Output**:
```
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
jenkins-service  NodePort   10.96.x.x       <none>        8080:30008/TCP   10s
```

---

## 🔹 Step 9: Verify Jenkins Accessibility

```bash
curl http://<node-ip>:30008
```

**Purpose**: Test the Jenkins login page.

**Expected Output**:
```
<html>...Jenkins...</html>
```

**Note**: Click the "Jenkins" button to verify in the lab environment.

---

## 📋 Quick Command Reference

```bash
# Create namespace and manifests
kubectl create namespace jenkins
kubectl config set-context --current --namespace=jenkins
vi jenkins-deployment.yaml
vi jenkins-service.yaml
# (Paste YAML contents)
kubectl apply -f jenkins-deployment.yaml
kubectl apply -f jenkins-service.yaml

# Verify resources
kubectl get deployment
kubectl get pods
kubectl get svc
curl http://<node-ip>:30008
```

---

## 💡 Additional Tips

- **Task Variability**: Namespace or nodePort may differ; adjust YAML as needed.
- **Jenkins Startup**: May take time to initialize; monitor pod status.
- **Access Control**: Secure Jenkins in production with authentication.
- **Dry Run**: Test with `kubectl apply -f <file>.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod jenkins-deployment-xxx-xxx
kubectl logs jenkins-deployment-xxx-xxx
```

### **Issue 2: Service Inaccessible**
**Symptoms**: `curl` fails on nodePort 30008.
**Solution**: Verify service configuration.
```bash
kubectl describe svc jenkins-service
# Ensure nodePort: 30008 and selector: app=jenkins
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod jenkins-deployment-xxx-xxx
# Ensure image: jenkins/jenkins
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying Jenkins** and ensuring the login page is accessible via a NodePort service.

**💡 Solution Approach:**
1. **Namespace Setup**: Created `jenkins` namespace.
2. **Deployment Configuration**: Used `jenkins/jenkins` with 1 replica.
3. **Service Exposure**: Configured nodePort 30008.
4. **Verification**: Confirmed pod status and Jenkins accessibility.

**🎯 Key Success Factors:**
- **Correct image**: Used `jenkins/jenkins`.
- **Service access**: NodePort 30008 exposes Jenkins.
- **Pod status**: Ensured `Running` state.
- **Namespace isolation**: Used `jenkins` namespace.

**⚠️ Critical Configuration Details:**
- **Namespace**: Must be `jenkins`.
- **Deployment Name**: Must be `jenkins-deployment`.
- **Container Name**: Must be `jenkins-container`.
- **Image**: Must be `jenkins/jenkins`.
- **Service**: `jenkins-service` with nodePort 30008.
- **Port**: Container port 8080.

**🔒 Kubernetes Benefits:**
- **CI/CD Integration**: Jenkins runs seamlessly in Kubernetes.
- **Exposability**: NodePort enables external access.
- **Isolation**: Namespaces separate workloads.

---

## ⚠️ Important Production Notes

🔧 **Jenkins Configuration**: Secure with authentication and RBAC.

🔐 **Security**: Use PersistentVolumes for Jenkins data.

📊 **Monitoring**: Monitor Jenkins pod health and resource usage.

🛡️ **Scalability**: Consider scaling Jenkins for large pipelines.

---
