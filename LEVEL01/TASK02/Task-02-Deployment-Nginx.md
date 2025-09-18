**🌟 Task 2 - Create Nginx Deployment**

**📌 Task Description**

The Nautilus DevOps team is advancing its Kubernetes adoption by deploying applications using deployments. A team member has been tasked with creating a deployment to test application scaling.

**Requirements:**
- Create a deployment named `nginx` using the `nginx:latest` image (specify the tag).
- Ensure the deployment is properly labeled and configured.

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster. Values like deployment names or namespaces may differ in your environment, but the core challenge remains the same.

👉 **Your task**: Deploy an nginx application using a Kubernetes deployment.

---

## 🔹 Step 1: Connect to Jump Host

```bash
ssh tony@jumphost
```

**Purpose**: Access the jump host where `kubectl` is configured.

---

## 🔹 Step 2: Create Deployment Manifest

```bash
vi nginx-deployment.yaml
```

**Purpose**: Create a YAML file to define the deployment configuration.

**YAML Content**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Command Breakdown**:
- `apiVersion: apps/v1`: Uses the `apps/v1` API for deployments.
- `kind: Deployment`: Defines a deployment resource.
- `metadata.name`: Sets deployment name to `nginx`.
- `spec.replicas`: Sets one replica (adjustable).
- `spec.selector`: Matches pods with `app: nginx` label.
- `spec.template`: Defines the pod template with container `nginx` and image `nginx:latest`.
- `imagePullPolicy: IfNotPresent`: Optimizes image pulling.
- `restartPolicy: Always`: Ensures pod restarts on failure.

---

## 🔹 Step 3: Apply the Deployment Manifest

```bash
kubectl apply -f nginx-deployment.yaml
```

**Purpose**: Deploy the nginx application to the cluster.

**Expected Output**:
```
deployment.apps/nginx created
```

---

## 🔹 Step 4: Verify Deployment

```bash
kubectl get deployments
```

**Purpose**: Confirm the deployment is created and running.

**Expected Output**:
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           10s
```

**Verify Pods**:
```bash
kubectl get pods -l app=nginx
```

**Expected Output**:
```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-xxx-xxx   1/1     Running   0          10s
```

---

## 🔹 Step 5: Verify Deployment Details

```bash
kubectl describe deployment nginx
```

**Purpose**: Check deployment details, including labels and container status.

**Expected Output (partial)**:
```
Name:                   nginx
Selector:               app=nginx
Replicas:               1 desired | 1 updated | 1 total
Containers:
  nginx:
    Image:        nginx:latest
    State:        Running
```

---

## 📋 Quick Command Reference

```bash
# Create and apply deployment manifest
vi nginx-deployment.yaml
# (Paste YAML content)
kubectl apply -f nginx-deployment.yaml

# Verify deployment and pods
kubectl get deployments
kubectl get pods -l app=nginx
kubectl describe deployment nginx
```

---

## 💡 Additional Tips

- **Task Variability**: Deployment names or namespaces may differ; adapt the YAML accordingly.
- **Replicas**: Default is 1 replica; scale as needed with `kubectl scale`.
- **Labels**: Consistent labeling (`app: nginx`) ensures proper selector matching.
- **Dry Run**: Use `kubectl apply -f nginx-deployment.yaml --dry-run=client` to test.

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Deployment Not Creating Pods**
**Symptoms**: `kubectl get pods` shows no pods.
**Solution**: Check deployment status and events.
```bash
kubectl describe deployment nginx
# Verify selector matches pod labels
```

### **Issue 2: Image Pull Failure**
**Symptoms**: Pods in `ErrImagePull` state.
**Solution**: Confirm image name and connectivity.
```bash
kubectl describe pod nginx-xxx-xxx
# Ensure image: nginx:latest
```

### **Issue 3: Selector Mismatch**
**Symptoms**: Pods created but not managed by deployment.
**Solution**: Verify `selector.matchLabels` matches `template.metadata.labels`.
```bash
kubectl get deployment nginx -o yaml
# Fix labels if mismatched
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **ensuring correct deployment configuration** with proper labeling and image specification to manage a single nginx replica.

**💡 Solution Approach:**
1. **Accurate YAML**: Defined deployment with `nginx` name, `nginx:latest` image, and consistent labels.
2. **Label Matching**: Ensured `selector.matchLabels` and `template.metadata.labels` both use `app: nginx`.
3. **Verification**: Used `kubectl get` and `describe` to confirm deployment and pod status.
4. **Best Practices**: Included `imagePullPolicy: IfNotPresent` and `restartPolicy: Always`.

**🎯 Key Success Factors:**
- **Correct naming**: Deployment and container named `nginx`.
- **Label consistency**: `app: nginx` for selector and pod template.
- **Single replica**: Set `replicas: 1` for initial deployment.
- **Verification**: Confirmed deployment and pod readiness.

**⚠️ Critical Configuration Details:**
- **Deployment Name**: Must be `nginx`.
- **Image**: Must be `nginx:latest`.
- **Labels**: Must include `app: nginx` for selector and pod.
- **Cluster Access**: Assumes `kubectl` is configured.

**🔒 Kubernetes Benefits:**
- **Scalability**: Deployments allow easy scaling with `kubectl scale`.
- **Reliability**: Automatic pod restarts and updates.
- **Manageability**: Labels enable service integration.

---

## ⚠️ Important Production Notes

🔧 **Deployment**: Manages nginx pods with automatic scaling and updates.

🔐 **Security**: Use specific image tags in production; avoid `latest`.

📊 **Monitoring**: Monitor pod resource usage with tools like Prometheus.

🛡️ **Scalability**: Adjust replicas for load balancing in production.

---