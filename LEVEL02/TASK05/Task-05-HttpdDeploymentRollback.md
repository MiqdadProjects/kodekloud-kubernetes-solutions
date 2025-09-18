**🌟 Task 5 - Deploy Httpd with Update and Rollback**

**📌 Task Description**

The Nautilus DevOps team is preparing for a production deployment by testing updates and rollbacks in a Dev environment.

**Requirements:**
- Create namespace `nautilus`.
- Create deployment `httpd-deploy` in `nautilus` namespace, using `httpd:2.4.27`, named `httpd`, 3 replicas, `RollingUpdate` strategy with `maxSurge: 1`, `maxUnavailable: 2`.
- Create `NodePort` service `httpd-service` in `nautilus` namespace, nodePort `30008`.
- Upgrade deployment to `httpd:2.4.43` using rolling update.
- Roll back to `httpd:2.4.27`.
- Ensure application is accessible via "Apache" button.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Use only specified images in sequence to maintain revision history.

👉 **Your task**: Deploy, update, and rollback an httpd application.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Create Namespace

```bash
kubectl create namespace nautilus
```

**Purpose**: Create the `nautilus` namespace.

**Expected Output**:
```
namespace/nautilus created
```

---

## 🔹 Step 3: Create Deployment and Service Manifest

```bash
vi nautilus-deployment.yaml
```

**Purpose**: Define the deployment and service.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
  namespace: nautilus
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  selector:
    matchLabels:
      app: httpd-app
  template:
    metadata:
      labels:
        app: httpd-app
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.27
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: nautilus
spec:
  type: NodePort
  selector:
    app: httpd-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30008
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `kind: Deployment`: Defines deployment with `RollingUpdate` strategy.
- `replicas: 3`: Ensures 3 pods.
- `strategy`: Sets `maxSurge: 1`, `maxUnavailable: 2`.
- `kind: Service`: Exposes deployment on nodePort 30008.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.

---

## 🔹 Step 4: Apply the Manifest

```bash
kubectl apply -f nautilus-deployment.yaml -n nautilus
```

**Purpose**: Deploy the resources.

**Expected Output**:
```
deployment.apps/httpd-deploy created
service/httpd-service created
```

---

## 🔹 Step 5: Verify Deployment and Service

```bash
kubectl get deployment -n nautilus
kubectl get svc -n nautilus
```

**Purpose**: Confirm resources are created.

**Expected Output**:
```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
httpd-deploy   3/3     3            3           10s

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
httpd-service   NodePort   10.96.x.x       <none>        80:30008/TCP   10s
```

---

## 🔹 Step 6: Verify Application Accessibility

```bash
curl http://<node-ip>:30008
```

**Purpose**: Test the httpd application.

**Expected Output**:
```
<html><body><h1>It works!</h1></body></html>
```

**Note**: Click the "Apache" button to verify in the lab environment.

---

## 🔹 Step 7: Upgrade Deployment

```bash
kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n nautilus
kubectl rollout status deployment/httpd-deploy -n nautilus
```

**Purpose**: Update to `httpd:2.4.43` and monitor rollout.

**Expected Output**:
```
deployment.apps/httpd-deploy image updated
deployment "httpd-deploy" successfully rolled out
```

---

## 🔹 Step 8: Verify Upgrade

```bash
kubectl describe deployment httpd-deploy -n nautilus
```

**Purpose**: Confirm image updated to `httpd:2.4.43`.

**Expected Output (partial)**:
```
Containers:
  httpd:
    Image:        httpd:2.4.43
```

---

## 🔹 Step 9: Roll Back Deployment

```bash
kubectl rollout undo deployment/httpd-deploy -n nautilus
kubectl rollout status deployment/httpd-deploy -n nautilus
```

**Purpose**: Revert to `httpd:2.4.27` and monitor rollback.

**Expected Output**:
```
deployment.apps/httpd-deploy rolled back
deployment "httpd-deploy" successfully rolled out
```

---

## 🔹 Step 10: Verify Rollback

```bash
kubectl describe deployment httpd-deploy -n nautilus
```

**Purpose**: Confirm image reverted to `httpd:2.4.27`.

**Expected Output (partial)**:
```
Containers:
  httpd:
    Image:        httpd:2.4.27
```

---

## 📋 Quick Command Reference

```bash
# Create namespace and apply manifest
kubectl create namespace nautilus
vi nautilus-deployment.yaml
# (Paste YAML content)
kubectl apply -f nautilus-deployment.yaml -n nautilus

# Verify resources
kubectl get deployment -n nautilus
kubectl get svc -n nautilus
curl http://<node-ip>:30008

# Update and rollback
kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n nautilus
kubectl rollout status deployment/httpd-deploy -n nautilus
kubectl rollout undo deployment/httpd-deploy -n nautilus
kubectl rollout status deployment/httpd-deploy -n nautilus
```

---

## 💡 Additional Tips

- **Task Variability**: Namespace, image versions, or nodePorts may differ; adjust YAML as needed.
- **Rolling Updates**: Ensure `maxSurge` and `maxUnavailable` balance availability.
- **Rollback**: Maintain revision history by using correct images.
- **Dry Run**: Test with `kubectl apply -f nautilus-deployment.yaml --dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Pods Not Updating**
**Symptoms**: Pods not updating to `httpd:2.4.43`.
**Solution**: Check rollout status and events.
```bash
kubectl rollout status deployment/httpd-deploy -n nautilus
kubectl describe pod httpd-deploy-xxx-xxx -n nautilus
```

### **Issue 2: Service Inaccessible**
**Symptoms**: `curl` fails on nodePort 30008.
**Solution**: Verify service configuration.
```bash
kubectl describe svc httpd-service -n nautilus
# Ensure nodePort: 30008 and selector: app=httpd-app
```

### **Issue 3: Rollback Fails**
**Symptoms**: Rollback does not revert to `httpd:2.4.27`.
**Solution**: Check revision history.
```bash
kubectl rollout history deployment/httpd-deploy -n nautilus
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **managing a rolling update and rollback** while maintaining service availability.

**💡 Solution Approach:**
1. **Deployment Setup**: Configured 3 replicas with `RollingUpdate` strategy.
2. **Service Exposure**: Used `NodePort` 30008 for access.
3. **Update and Rollback**: Performed rolling update to `httpd:2.4.43` and reverted.
4. **Verification**: Confirmed pod status and accessibility.

**🎯 Key Success Factors:**
- **Correct images**: Used `httpd:2.4.27` and `httpd:2.4.43` in sequence.
- **Rolling update**: Maintained availability with `maxSurge: 1`, `maxUnavailable: 2`.
- **Rollback success**: Reverted to original image.
- **Service access**: Ensured application availability.

**⚠️ Critical Configuration Details:**
- **Namespace**: Must be `nautilus`.
- **Deployment Name**: Must be `httpd-deploy`.
- **Container Name**: Must be `httpd`.
- **Images**: Must use `httpd:2.4.27` then `httpd:2.4.43`.
- **Service**: `httpd-service` with nodePort 30008.
- **Strategy**: `RollingUpdate` with `maxSurge: 1`, `maxUnavailable: 2`.

**🔒 Kubernetes Benefits:**
- **Zero Downtime**: Rolling updates ensure continuous availability.
- **Rollback**: Revision history enables quick recovery.
- **Scalability**: Multiple replicas enhance reliability.

---

## ⚠️ Important Production Notes

🔧 **Update Strategy**: Test updates in staging environments.

🔐 **Security**: Use specific image tags and scan for vulnerabilities.

📊 **Monitoring**: Monitor rollout status and application performance.

🛡️ **Rollback Plan**: Maintain revision history for quick recovery.

---
