**🌟 Task 12 - Update Deployment and Service**

**📌 Task Description**

The Nautilus application development team has developed a new version of an application, requiring updates to an existing Kubernetes deployment and service. The changes must be applied without deleting the resources.

**Requirements:**
- Update the `nginx-deployment` deployment and `nginx-service` service.
- Change the service `nodePort` from `30008` to `32165`.
- Change the deployment replicas from 1 to 5.
- Change the container image from `nginx:1.19` to `nginx:latest`.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like ports or image versions may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Update the deployment and service with new configurations.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host for `kubectl` commands.

---

## 🔹 Step 2: Verify Existing Resources

```bash
kubectl get deployment nginx-deployment
kubectl get svc nginx-service
```

**Purpose**: Confirm the deployment and service exist.

**Expected Output**:
```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment  1/1     1            1           1h

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.x.x       <none>        80:30008/TCP   1h
```

---

## 🔹 Step 3: Update Service NodePort

```bash
kubectl patch svc nginx-service --type='json' -p '[{"op":"replace","path":"/spec/ports/0/nodePort","value":32165}]'
```

**Purpose**: Change the `nodePort` to `32165`.

**Expected Output**:
```
service/nginx-service patched
```

**Command Breakdown**:
- `kubectl patch`: Updates specific fields in the service.
- `--type='json'`: Uses JSON patch format.
- `op: replace`: Replaces the `nodePort` value.
- `path: /spec/ports/0/nodePort`: Targets the first port’s `nodePort`.

---

## 🔹 Step 4: Update Deployment Replicas and Image

```bash
kubectl patch deployment nginx-deployment --type='json' -p '[{"op":"replace","path":"/spec/replicas","value":5},{"op":"replace","path":"/spec/template/spec/containers/0/image","value":"nginx:latest"}]'
```

**Purpose**: Update replicas to 5 and image to `nginx:latest`.

**Expected Output**:
```
deployment.apps/nginx-deployment patched
```

**Command Breakdown**:
- `op: replace`: Updates `replicas` and `image` fields.
- `path: /spec/replicas`: Sets replica count to 5.
- `path: /spec/template/spec/containers/0/image`: Updates the first container’s image.

**Alternative Approach**:
```bash
kubectl edit deployment nginx-deployment
# Change replicas: 1 to replicas: 5
# Change image: nginx:1.19 to image: nginx:latest
# Save and exit
```

---

## 🔹 Step 5: Monitor Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

**Purpose**: Confirm the deployment update completes successfully.

**Expected Output**:
```
deployment "nginx-deployment" successfully rolled out
```

---

## 🔹 Step 6: Verify Updated Resources

```bash
kubectl get deployment nginx-deployment
kubectl get svc nginx-service
```

**Purpose**: Confirm updated replicas and nodePort.

**Expected Output**:
```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment  5/5     5            5           1h

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.x.x       <none>        80:32165/TCP   1h
```

**Verify Pods**:
```bash
kubectl get pods -l app=nginx
```

**Expected Output**:
```
NAME                             READY   STATUS    RESTARTS   AGE
nginx-deployment-xxx-xxx   1/1     Running   0          10s
...
```

**Verify Image**:
```bash
kubectl describe deployment nginx-deployment
```

**Expected Output (partial)**:
```
Containers:
  nginx:
    Image:        nginx:latest
```

---

## 🔹 Step 7: Verify Application Accessibility

```bash
curl http://<node-ip>:32165
```

**Purpose**: Confirm the application is accessible on the new nodePort.

**Expected Output**:
```
Welcome to nginx!
```

---

## 📋 Quick Command Reference

```bash
# Verify resources
kubectl get deployment nginx-deployment
kubectl get svc nginx-service

# Update service and deployment
kubectl patch svc nginx-service --type='json' -p '[{"op":"replace","path":"/spec/ports/0/nodePort","value":32165}]'
kubectl patch deployment nginx-deployment --type='json' -p '[{"op":"replace","path":"/spec/replicas","value":5},{"op":"replace","path":"/spec/template/spec/containers/0/image","value":"nginx:latest"}]'

# Verify updates
kubectl rollout status deployment/nginx-deployment
kubectl get deployment nginx-deployment
kubectl get svc nginx-service
kubectl get pods -l app=nginx
curl http://<node-ip>:32165
```

---

## 💡 Additional Tips

- **Task Variability**: NodePort, replicas, or image versions may differ; verify existing configurations.
- **Patch Efficiency**: `kubectl patch` minimizes manual edits.
- **Rollback Option**: Use `kubectl rollout undo` if updates fail.
- **Dry Run**: Test patches with `--dry-run=client`.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Patch Fails**
**Symptoms**: `kubectl patch` errors out.
**Solution**: Validate JSON patch syntax.
```bash
kubectl patch svc nginx-service --type='json' -p '[{"op":"replace","path":"/spec/ports/0/nodePort","value":32165}]' --dry-run=client
```

### **Issue 2: Pods Not Scaling**
**Symptoms**: Fewer than 5 pods running.
**Solution**: Check deployment events.
```bash
kubectl describe deployment nginx-deployment
```

### **Issue 3: Service Inaccessible**
**Symptoms**: `curl` fails on new nodePort.
**Solution**: Verify nodePort and cluster connectivity.
```bash
kubectl describe svc nginx-service
# Ensure nodePort: 32165
```

### **Issue 4: Image Pull Failure**
**Symptoms**: Pods in `ErrImagePull`.
**Solution**: Verify image.
```bash
kubectl describe pod nginx-deployment-xxx-xxx
# Ensure image: nginx:latest
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **updating multiple fields** in a deployment and service without deleting them, ensuring zero downtime.

**💡 Solution Approach:**
1. **Patch Commands**: Used `kubectl patch` for precise updates to nodePort, replicas, and image.
2. **Rolling Update**: Ensured updates applied without downtime.
3. **Verification**: Confirmed pod scaling, image update, and service accessibility.
4. **Best Practices**: Used efficient `patch` commands and monitored rollout status.

**🎯 Key Success Factors:**
- **Accurate updates**: Changed nodePort to 32165, replicas to 5, image to `nginx:latest`.
- **No downtime**: Rolling update maintained availability.
- **Verification**: Confirmed all changes and accessibility.
- **Efficient approach**: Used `kubectl patch` for minimal disruption.

**⚠️ Critical Configuration Details:**
- **Deployment Name**: Must be `nginx-deployment`.
- **Service Name**: Must be `nginx-service`.
- **NodePort**: Must be `32165`.
- **Replicas**: Must be `5`.
- **Image**: Must be `nginx:latest`.

**🔒 Kubernetes Benefits:**
- **Scalability**: Increased replicas enhance availability.
- **Flexibility**: `kubectl patch` allows precise updates.
- **Zero Downtime**: Rolling updates ensure continuous service.

---

## ⚠️ Important Production Notes

🔧 **Deployment Updates**: Rolling updates minimize downtime.

🔐 **Security**: Verify new image versions for security patches.

📊 **Monitoring**: Monitor pod scaling and service performance.

🛡️ **Rollback**: Keep previous configurations for quick rollback.

---
