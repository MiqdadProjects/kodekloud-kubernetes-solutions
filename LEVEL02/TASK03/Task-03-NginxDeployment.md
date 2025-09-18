**🌟 Task 3 - Create Nginx Deployment and Service**

**📌 Task Description**

The Nautilus team is developing a static website and wants to deploy it on a Kubernetes cluster with high availability and scalability.

**Requirements:**
- Create a deployment named `nginx-deployment` with `nginx:latest` image, container named `nginx-container`, 3 replicas.
- Create a `NodePort` service named `nginx-service` with `nodePort: 30011`.
- Ensure all pods are running and the application is accessible via the "App" button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like names or ports may differ in your environment.

👉 **Your task**: Deploy a scalable nginx application with a NodePort service.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Deployment and Service Manifest

```bash
vi nginx-deployment.yaml
```

**Purpose**: Define the deployment and service.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30011
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Deployment`: Defines a deployment with 3 replicas.
- `selector.matchLabels`: Matches pods with `app: nginx`.
- `containers`: Uses `nginx:latest` with port 80.
- `kind: Service`: Exposes deployment as `NodePort` on port 30011.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply the Manifest

```bash
kubectl apply -f nginx-deployment.yaml
```

**Purpose**: Deploy the deployment and service.

**Expected Output**:
```
deployment.apps/nginx-deployment created
service/nginx-service created
```

---

## 🔹 Step 4: Verify Deployment

```bash
kubectl get deployment
```

**Purpose**: Confirm 3 replicas are ready.

**Expected Output**:
```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment  3/3     3            3           10s
```

---

## 🔹 Step 5: Verify Pods

```bash
kubectl get pods -l app=nginx
```

**Purpose**: Ensure all pods are running.

**Expected Output**:
```
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-xxx-xxx   1/1     Running   0          10s
nginx-deployment-xxx-xxx   1/1     Running   0          10s
nginx-deployment-xxx-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 6: Verify Service

```bash
kubectl get svc
```

**Purpose**: Confirm the service is configured.

**Expected Output**:
```
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.x.x       <none>        80:30011/TCP   10s
```

---

## 🔹 Step 7: Verify Application Accessibility

```bash
curl http://<node-ip>:30011
```

**Purpose**: Test the nginx application via the NodePort.

**Expected Output**:
```
<html>
<head><title>Welcome to nginx!</title></head>
...
```

**Note**: Click the "App" button to verify accessibility in the lab environment.

---

## 📋 Quick Command Reference

```bash
# Create and apply manifest
vi nginx-deployment.yaml
# (Paste YAML content)
kubectl apply -f nginx-deployment.yaml

# Verify resources
kubectl get deployment
kubectl get pods -l app=nginx
kubectl get svc
curl http://<node-ip>:30011
```

---

## 💡 Additional Tips

- **Task Variability**: Deployment names or nodePorts may differ; adjust YAML as needed.
- **Scalability**: Use Deployments for high availability.
- **Service Access**: Ensure nodePort is accessible in the cluster.
- **Dry Run**: Test with `kubectl apply -f nginx-deployment.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pods Not Running**
**Symptoms**: Pods in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events.
```bash
kubectl describe pod nginx-deployment-xxx-xxx
kubectl logs nginx-deployment-xxx-xxx
```

### **Issue 2: Service Inaccessible**
**Symptoms**: `curl` fails on nodePort 30011.
**Solution**: Verify service configuration.
```bash
kubectl describe svc nginx-service
# Ensure nodePort: 30011 and selector: app=nginx
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod nginx-deployment-xxx-xxx
# Ensure image: nginx:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying a scalable nginx application** with a NodePort service for external access.

**💡 Solution Approach:**
1. **Deployment Setup**: Configured 3 replicas with `nginx:latest`.
2. **Service Configuration**: Exposed deployment on nodePort 30011.
3. **Verification**: Confirmed pod status and application accessibility.
4. **Best Practices**: Used consistent labels and efficient YAML.

**🎯 Key Success Factors:**
- **Replica count**: 3 pods running.
- **Service access**: NodePort 30011 exposes nginx.
- **Pod status**: All pods in `Running` state.
- **Efficient setup**: Combined deployment and service in one YAML.

**⚠️ Critical Configuration Details:**
- **Deployment Name**: Must be `nginx-deployment`.
- **Container Name**: Must be `nginx-container`.
- **Image**: Must be `nginx:latest`.
- **Replicas**: Must be 3.
- **Service**: `nginx-service` with nodePort 30011.

**🔒 Kubernetes Benefits:**
- **Scalability**: Deployments manage replica counts.
- **High Availability**: Multiple pods ensure redundancy.
- **Exposability**: NodePort enables external access.

---

## ⚠️ Important Production Notes

🔧 **Deployment Strategy**: Use RollingUpdate for zero-downtime updates.

🔐 **Security**: Restrict nodePort access with network policies.

📊 **Monitoring**: Monitor pod health and traffic.

🛡️ **Scalability**: Adjust replica counts based on load.

---