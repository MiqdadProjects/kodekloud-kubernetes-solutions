**🌟 Task 7 - Deploy Grafana with NodePort Service**

**📌 Task Description**

The Nautilus DevOps team is setting up Grafana to collect and analyze application analytics on a Kubernetes cluster.

**Requirements:**
- Create deployment `grafana-deployment-xfusion` with `grafana/grafana:latest` image.
- Create `NodePort` service with nodePort `32000`.
- Ensure Grafana login page is accessible via the "Grafana" button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like names or ports may differ in your environment.

👉 **Your task**: Deploy Grafana and expose it via a NodePort service.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Deployment and Service Manifest

```bash
vi grafana-deployment.yaml
```

**Purpose**: Define the Grafana deployment and service.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-xfusion
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana-container
        image: grafana/grafana:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  labels:
    app: grafana
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
    nodePort: 32000
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Deployment`: Defines Grafana deployment with 1 replica.
- `labels` and `selector`: Uses `app: grafana` for matching.
- `kind: Service`: Exposes deployment on nodePort 32000.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 3: Apply Manifests

```bash
kubectl apply -f grafana-deployment.yaml
```

**Purpose**: Deploy the resources.

**Expected Output**:
```
deployment.apps/grafana-deployment-xfusion created
service/grafana-service created
```

---

## 🔹 Step 4: Verify Deployment

```bash
kubectl get deployment
```

**Purpose**: Confirm deployment is ready.

**Expected Output**:
```
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
grafana-deployment-xfusion  1/1     1            1           10s
```

---

## 🔹 Step 5: Verify Pods

```bash
kubectl get pods
```

**Purpose**: Ensure pod is running.

**Expected Output**:
```
NAME                                         READY   STATUS    RESTARTS   AGE
grafana-deployment-xfusion-xxx-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 6: Verify Service

```bash
kubectl get svc
```

**Purpose**: Confirm service configuration.

**Expected Output**:
```
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana-service   NodePort   10.96.x.x       <none>        3000:32000/TCP   10s
```

---

## 🔹 Step 7: Verify Grafana Accessibility

```bash
curl http://<node-ip>:32000
```

**Purpose**: Test the Grafana login page.

**Expected Output**:
```
<html>...Grafana...</html>
```

**Note**: Click the "Grafana" button to verify in the lab environment.

---

## 📋 Quick Command Reference

```bash
# Create and apply manifest
vi grafana-deployment.yaml
# (Paste YAML content)
kubectl apply -f grafana-deployment.yaml

# Verify resources
kubectl get deployment
kubectl get pods
kubectl get svc
curl http://<node-ip>:32000
```

---

## 💡 Additional Tips

- **Task Variability**: Deployment names or nodePorts may differ; adjust YAML as needed.
- **Grafana Startup**: May take time to initialize; monitor pod status.
- **Access Control**: Secure Grafana with authentication in production.
- **Dry Run**: Test with `kubectl apply -f grafana-deployment.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pod Not Running**
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.
**Solution**: Check pod events and logs.
```bash
kubectl describe pod grafana-deployment-xfusion-xxx-xxx
kubectl logs grafana-deployment-xfusion-xxx-xxx
```

### **Issue 2: Service Inaccessible**
**Symptoms**: `curl` fails on nodePort 32000.
**Solution**: Verify service configuration.
```bash
kubectl describe svc grafana-service
# Ensure nodePort: 32000 and selector: app=grafana
```

### **Issue 3: Image Pull Failure**
**Symptoms**: `ErrImagePull` status.
**Solution**: Verify image name.
```bash
kubectl describe pod grafana-deployment-xfusion-xxx-xxx
# Ensure image: grafana/grafana:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying Grafana** and ensuring the login page is accessible via a NodePort service.

**💡 Solution Approach:**
1. **Deployment Configuration**: Used `grafana/grafana:latest` with 1 replica.
2. **Service Exposure**: Configured nodePort 32000.
3. **Verification**: Confirmed pod status and Grafana accessibility.
4. **Best Practices**: Used consistent labels and minimal configuration.

**🎯 Key Success Factors:**
- **Correct image**: Used `grafana/grafana:latest`.
- **Service access**: NodePort 32000 exposes Grafana.
- **Pod status**: Ensured `Running` state.
- **Efficient setup**: Combined deployment and service in one YAML.

**⚠️ Critical Configuration Details:**
- **Deployment Name**: Must be `grafana-deployment-xfusion`.
- **Container Name**: Must be `grafana-container`.
- **Image**: Must be `grafana/grafana:latest`.
- **Service**: `grafana-service` with nodePort 32000.
- **Port**: Container port 3000.

**🔒 Kubernetes Benefits:**
- **Analytics Platform**: Grafana integrates with Kubernetes monitoring.
- **Exposability**: NodePort enables external access.
- **Flexibility**: Easy to scale or update.

---

## ⚠️ Important Production Notes

🔧 **Grafana Configuration**: Secure with authentication and persistent storage.

🔐 **Security**: Use Secrets for Grafana credentials.

📊 **Monitoring**: Monitor Grafana pod health and resource usage.

🛡️ **Scalability**: Consider scaling for large analytics workloads.

---
