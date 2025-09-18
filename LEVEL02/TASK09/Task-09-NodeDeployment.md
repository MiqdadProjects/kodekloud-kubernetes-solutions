**🌟 Task 9 - Deploy Node Application with NodePort Service**

**📌 Task Description**

The Nautilus development team has completed a Node.js application and needs it deployed on a Kubernetes cluster with specific requirements.

**Requirements:**
- Create deployment using `gcr.io/kodekloud/centos-ssh-enabled:node` image, 2 replicas.
- Create NodePort service with targetPort 8080, nodePort 30012.
- Ensure all pods are running and application is accessible via the "NodeApp" button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like names or ports may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy a Node.js application and expose it via a NodePort service.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Deployment and Service Manifest

```bash
vi node-deploy.yaml
```

**Purpose**: Define the Node.js deployment and service.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodeapp
  template:
    metadata:
      labels:
        app: nodeapp
    spec:
      containers:
      - name: node-container
        image: gcr.io/kodekloud/centos-ssh-enabled:node
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: node-service
spec:
  type: NodePort
  selector:
    app: nodeapp
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
    nodePort: 30012
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Deployment`: Defines deployment with 2 replicas.
- `selector.matchLabels`: Matches pods with `app: nodeapp`.
- `containers`: Uses `gcr.io/kodekloud/centos-ssh-enabled:node` with port 8080.
- `kind: Service`: Exposes deployment as `NodePort` on 30012.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply the Manifests

```bash
kubectl apply -f node-deploy.yaml
```

**Purpose**: Deploy the resources.

**Expected Output**:
```
deployment.apps/node-deployment created
service/node-service created
```

---

## 🔹 Step 4: Verify Deployment

```bash
kubectl get deployment
```

**Purpose**: Confirm the deployment is ready with 2 replicas.

**Expected Output**:
```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
node-deployment  2/2     2            2           10s
```

**Verification**: Ensure the deployment has 2 available pods.

---

## 🔹 Step 5: Verify Pods

```bash
kubectl get pods
```

**Purpose**: Ensure all pods are running.

**Expected Output**:
```
NAME                              READY   STATUS    RESTARTS   AGE
node-deployment-xxx-xxx   1/1     Running   0          10s
node-deployment-xxx-xxx   1/1     Running   0          10s
```

**Verification**: Both pods should be in `Running` state.

---

## 🔹 Step 6: Verify Service

```bash
kubectl get svc
```

**Purpose**: Confirm the service configuration.

**Expected Output**:
```
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
node-service   NodePort   10.96.x.x       <none>        8080:30012/TCP   10s
```

**Verification**: The service should have nodePort 30012 and targetPort 8080.

---

## 🔹 Step 7: Verify Node Application Accessibility

```bash
curl http://<node-ip>:30012
```

**Purpose**: Test the Node.js application.

**Expected Output**:
```
<html>...Everything Sharks...</html>
```

**Note**: Click the "NodeApp" button to verify in the lab environment, which should show the "Everything Sharks" page.

---

## 📋 Quick Command Reference

```bash
# Create and apply manifest
vi node-deploy.yaml
# (Paste YAML content)
kubectl apply -f node-deploy.yaml

# Verify resources
kubectl get deployment
kubectl get pods
kubectl get svc
curl http://<node-ip>:30012
```

---

## 💡 Additional Tips

- **Task Variability**: Deployment names or nodePorts may differ; adjust YAML as needed.
- **Application Startup**: The Node.js app may take time to initialize; monitor pod status.
- **Access Control**: Secure the application with authentication in production.
- **Dry Run**: Test with `kubectl apply -f node-deploy.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pods Not Running**
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod node-deployment-xxx-xxx
kubectl logs node-deployment-xxx-xxx
```

### **Issue 2: Service Inaccessible**
**Symptoms**: `curl` fails on nodePort 30012.
**Solution**: Verify service configuration.
```bash
kubectl describe svc node-service
# Ensure nodePort: 30012 and selector: app=nodeapp
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod node-deployment-xxx-xxx
# Ensure image: gcr.io/kodekloud/centos-ssh-enabled:node
```

### **Issue 4: Application Not Responding**
**Symptoms**: "NodeApp" button shows error or blank page.
**Solution**: Check application logs and service selector.
```bash
kubectl logs node-deployment-xxx-xxx
kubectl describe svc node-service
# Ensure targetPort: 8080 and selector matches deployment labels
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying a Node.js application** with 2 replicas and ensuring accessibility via a NodePort service.

**💡 Solution Approach:**
1. **Deployment Configuration**: Defined a deployment with 2 replicas using the specified image.
2. **Service Configuration**: Created a NodePort service with targetPort 8080 and nodePort 30012.
3. **Verification**: Confirmed pod status, service configuration, and application accessibility via "NodeApp" button.
4. **Best Practices**: Used consistent labels and optimized image pulling.

**🎯 Key Success Factors:**
- **Replica count**: 2 pods running.
- **Service access**: NodePort 30012 exposes the app.
- **Pod status**: All pods in `Running` state.
- **Application verification**: "Everything Sharks" page appears.

**⚠️ Critical Configuration Details:**
- **Deployment Image**: Must be `gcr.io/kodekloud/centos-ssh-enabled:node`.
- **Replicas**: Must be 2.
- **Service**: NodePort with targetPort 8080, nodePort 30012.
- **Labels**: Use `app: nodeapp` for selector and pod template.

**🔒 Kubernetes Benefits:**
- **Scalability**: Multiple replicas ensure high availability.
- **Exposability**: NodePort enables external access.
- **Manageability**: Deployments handle replica maintenance.

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use RollingUpdate for zero-downtime updates.

🔐 **Security**: Secure NodePort access with network policies.

📊 **Monitoring**: Monitor pod health and traffic.

🛡️ **Scalability**: Adjust replicas based on load.

---
