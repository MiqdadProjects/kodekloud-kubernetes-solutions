# 🌟 Task 3 - Deploy Nginx and PHP-FPM Application on Kubernetes Cluster

## 📌 Task Description

The Nautilus Application Development team plans to deploy a PHP-based application on a Kubernetes cluster using nginx and PHP-FPM. The DevOps team has finalized the requirements, including custom nginx configurations, a shared volume for nginx and PHP-FPM, and external access via a NodePort service. Your task is to create the required Kubernetes resources and ensure the application is accessible.

### Requirements:
1. **Service**: Create a NodePort service with `nodePort: 30012` to expose the application.
2. **ConfigMap**: Create `nginx-config` for `nginx.conf` with:
   - Default port changed from 80 to 8093.
   - Document root changed from `/usr/share/nginx` to `/var/www/html`.
   - Directory index set to `index index.html index.htm index.php`.
3. **Pod**: Create `nginx-phpfpm` with:
   - **Shared Volume**: An `emptyDir` volume named `shared-files` mounted at `/var/www/html` in both containers.
   - **Nginx Container**: Named `nginx-container`, using `nginx:latest`, with ConfigMap volume `nginx-config-volume` mounted at `/etc/nginx/nginx.conf` with `subPath: nginx.conf`.
   - **PHP-FPM Container**: Named `php-fpm-container`, using `php:8.3-fpm-alpine`.
4. **File Copy**: Copy `/opt/index.php` from the jump host to `/var/www/html/index.php` in the nginx container.
5. **Verification**: Ensure the pod is running, the service is accessible via the **App** button, and the application displays "It works!".

**Note**: The `kubectl` utility on the jump host is configured to work with the Kubernetes cluster.

👉 **Your task**: Create the ConfigMap, Pod, and Service configurations, apply them, copy the `index.php` file, and verify that the application is accessible.

---

## 🔧 Infrastructure Overview

**Target Environment**: Kubernetes Cluster  
**Resources**:
- ConfigMap: `nginx-config` for custom nginx configuration
- Pod: `nginx-phpfpm` with nginx and PHP-FPM containers
- Service: `nginx-phpfpm-service` to expose the application via NodePort
- Namespace: Default

**Working Directory**: Jump host with `kubectl` configured

---

## 📋 Solution Overview

### 🏗️ Architecture Components
- **ConfigMap**: Stores custom nginx configuration
- **Pod**: Runs nginx and PHP-FPM containers with a shared volume
- **Service**: Exposes the application on NodePort 30012
- **Application**: PHP-based application served by nginx and processed by PHP-FPM

### 🎯 Implementation Strategy
1. Create a ConfigMap with the custom nginx configuration.
2. Create a pod with nginx and PHP-FPM containers, including a shared `emptyDir` volume and ConfigMap volume.
3. Create a NodePort service to expose the application.
4. Apply the configurations using `kubectl`.
5. Copy the `index.php` file to the nginx container.
6. Verify the pod status, service accessibility, and application output.

---

## 🚨 Potential Errors to Avoid

1. **ConfigMap Misconfiguration**:
   **Issue**: Incorrect nginx configuration (e.g., wrong port, root, or index settings).
   **Fix**: Ensure `listen 8093`, `root /var/www/html`, and `index index.html index.htm index.php`.

2. **Volume Mount Issues**:
   **Issue**: Incorrect mount paths or missing `subPath` for ConfigMap.
   **Fix**: Verify `shared-files` is mounted at `/var/www/html` and `nginx-config-volume` at `/etc/nginx/nginx.conf` with `subPath: nginx.conf`.

3. **Service Port Mismatch**:
   **Issue**: Incorrect `nodePort`, `port`, or `targetPort`.
   **Fix**: Set `nodePort: 30012`, `port: 8093`, and `targetPort: 8093`.

4. **File Copy Failure**:
   **Issue**: `index.php` not copied correctly to the nginx container.
   **Fix**: Use `kubectl cp` with the correct container and path.

---

## 🚀 Implementation Steps

### Step 1: Connect to Jump Host
```bash
ssh thor@jumphost
```

**Purpose**: Access the jump host to execute `kubectl` commands.

---

### Step 2: Create Configuration File

#### Nginx and PHP-FPM Configuration (`nginx-phpfpm-task3.yaml`)
Create or edit a file named `nginx-phpfpm-task3.yaml`:

```yaml
# 1. ConfigMap for Nginx Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
data:
  nginx.conf: |
    events { }
    http {
      server {
        listen 8093;
        root /var/www/html;
        index index.html index.htm index.php;

        location / {
          try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
          include fastcgi_params;
          fastcgi_pass 127.0.0.1:9000;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
        }
      }
    }

---
# 2. Pod with Nginx and PHP-FPM
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  namespace: default
  labels:
    app: nginx-phpfpm
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 8093
      name: nginx-port
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
  - name: php-fpm-container
    image: php:8.3-fpm-alpine
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
  volumes:
  - name: shared-files
    emptyDir: {}
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: nginx.conf

---
# 3. Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-phpfpm-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: nginx-phpfpm
  ports:
  - protocol: TCP
    port: 8093
    targetPort: 8093
    nodePort: 30012
```

**Configuration Breakdown**:
- **ConfigMap**:
  - **Name**: `nginx-config`
  - **Namespace**: `default`
  - **Data**: `nginx.conf` with `listen 8093`, `root /var/www/html`, and `index index.html index.htm index.php`
- **Pod**:
  - **Name**: `nginx-phpfpm`
  - **Namespace**: `default`
  - **Labels**: `app: nginx-phpfpm`
  - **Containers**:
    - **Nginx Container**: `nginx-container`, image `nginx:latest`, port 8093
    - **PHP-FPM Container**: `php-fpm-container`, image `php:8.3-fpm-alpine`
  - **Volumes**:
    - `shared-files`: `emptyDir` mounted at `/var/www/html` in both containers
    - `nginx-config-volume`: ConfigMap `nginx-config` mounted at `/etc/nginx/nginx.conf` with `subPath: nginx.conf`
- **Service**:
  - **Name**: `nginx-phpfpm-service`
  - **Namespace**: `default`
  - **Type**: `NodePort`
  - **Selector**: `app: nginx-phpfpm`
  - **Ports**: `port: 8093`, `targetPort: 8093`, `nodePort: 30012`

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

### Step 3: Apply Configurations
```bash
kubectl apply -f nginx-phpfpm-task3.yaml
```

**Purpose**: Create the ConfigMap, Pod, and Service in the Kubernetes cluster.

**Expected Output**:
```
configmap/nginx-config created
pod/nginx-phpfpm created
service/nginx-phpfpm-service created
```

---

### Step 4: Copy index.php File
```bash
kubectl cp /opt/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

**Purpose**: Copy the `index.php` file from the jump host to the nginx container’s document root.

---

### Step 5: Verify Resources
```bash
kubectl get configmap nginx-config
```

**Expected Output**:
```
NAME           DATA   AGE
nginx-config   1      2m
```

```bash
kubectl get pod nginx-phpfpm
```

**Expected Output**:
```
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          2m
```

```bash
kubectl get svc nginx-phpfpm-service
```

**Expected Output**:
```
NAME                  TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx-phpfpm-service  NodePort   10.96.123.789   <none>        8093:30012/TCP   2m
```

```bash
kubectl describe svc nginx-phpfpm-service
```

**Expected Output**:
```
Name:                     nginx-phpfpm-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx-phpfpm
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.789
IPs:                      10.96.123.789
Port:                     <unset>  8093/TCP
TargetPort:               8093/TCP
NodePort:                 <unset>  30012/TCP
Endpoints:                10.244.0.8:8093
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
curl http://<node-ip>:30012
```

**Expected Output**:
```
It works!
```

**Purpose**: Confirm the application is accessible and displays "It works!".

---

### Step 7: Verify ConfigMap Mount
```bash
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf
```

**Expected Output (Excerpt)**:
```
events { }
http {
  server {
    listen 8093;
    root /var/www/html;
    index index.html index.htm index.php;
    ...
```

**Purpose**: Confirm the ConfigMap is correctly mounted with the custom nginx configuration.

---

### Step 8: Verify Shared Volume
```bash
kubectl exec nginx-phpfpm -c nginx-container -- ls /var/www/html
```

**Expected Output**:
```
index.php
```

**Purpose**: Confirm the `index.php` file is in the shared volume.

---

## 🔍 Code Analysis

### Nginx and PHP-FPM Configuration (`nginx-phpfpm-task3.yaml`)
```yaml
# 1. ConfigMap for Nginx Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
data:
  nginx.conf: |
    events { }
    http {
      server {
        listen 8093;
        root /var/www/html;
        index index.html index.htm index.php;

        location / {
          try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
          include fastcgi_params;
          fastcgi_pass 127.0.0.1:9000;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
        }
      }
    }

---
# 2. Pod with Nginx and PHP-FPM
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  namespace: default
  labels:
    app: nginx-phpfpm
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 8093
      name: nginx-port
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
  - name: php-fpm-container
    image: php:8.3-fpm-alpine
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
  volumes:
  - name: shared-files
    emptyDir: {}
  - name: nginx-config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: nginx.conf

---
# 3. Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-phpfpm-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: nginx-phpfpm
  ports:
  - protocol: TCP
    port: 8093
    targetPort: 8093
    nodePort: 30012
```

### Resource Attributes
| Resource | Attribute | Value | Description |
|----------|-----------|-------|-------------|
| **ConfigMap** | nginx-config | nginx.conf | Custom nginx configuration |
| **Pod** | nginx-phpfpm | app: nginx-phpfpm | Runs nginx and PHP-FPM |
| **Nginx Container** | nginx-container | image: nginx:latest, port: 8093 | Serves web content |
| **PHP-FPM Container** | php-fpm-container | image: php:8.3-fpm-alpine | Processes PHP files |
| **Volume (shared-files)** | shared-files | emptyDir | Mounted at /var/www/html |
| **Volume (nginx-config-volume)** | nginx-config-volume | ConfigMap: nginx-config | Mounted at /etc/nginx/nginx.conf |
| **Service** | nginx-phpfpm-service | type: NodePort | Exposes application externally |
| **Service Ports** | port, targetPort, nodePort | 8093, 8093, 30012 | Routes traffic to nginx |

---

## ✅ Verification Steps

### Step 1: Verify ConfigMap
```bash
kubectl get configmap nginx-config
```

**Expected Output**:
```
NAME           DATA   AGE
nginx-config   1      2m
```

### Step 2: Verify Pod
```bash
kubectl get pod nginx-phpfpm
```

**Expected Output**:
```
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          2m
```

### Step 3: Verify Service
```bash
kubectl get svc nginx-phpfpm-service
```

**Expected Output**:
```
NAME                  TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
nginx-phpfpm-service  NodePort   10.96.123.789   <none>        8093:30012/TCP   2m
```

### Step 4: Verify Service Endpoints
```bash
kubectl describe svc nginx-phpfpm-service
```

**Expected Output**:
```
Name:                     nginx-phpfpm-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx-phpfpm
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.123.789
IPs:                      10.96.123.789
Port:                     <unset>  8093/TCP
TargetPort:               8093/TCP
NodePort:                 <unset>  30012/TCP
Endpoints:                10.244.0.8:8093
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Step 5: Verify ConfigMap Mount
```bash
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf
```

**Expected Output (Excerpt)**:
```
events { }
http {
  server {
    listen 8093;
    root /var/www/html;
    index index.html index.htm index.php;
    ...
```

### Step 6: Verify Shared Volume
```bash
kubectl exec nginx-phpfpm -c nginx-container -- ls /var/www/html
```

**Expected Output**:
```
index.php
```

### Step 7: Verify Application
```bash
curl http://<node-ip>:30012
```

**Expected Output**:
```
It works!
```

---

## 🧪 Testing

### Verify Nginx Port
```bash
kubectl exec nginx-phpfpm -c nginx-container -- netstat -tuln
```

**Expected Output**:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:8093            0.0.0.0:*               LISTEN
```

**Purpose**: Confirm nginx is listening on port 8093.

### Verify PHP-FPM
```bash
kubectl exec nginx-phpfpm -c php-fpm-container -- ps aux
```

**Expected Output (Excerpt)**:
```
PID   USER     TIME  COMMAND
    1 root      0:00 php-fpm: master process
```

**Purpose**: Confirm PHP-FPM is running.

### Verify File Accessibility
```bash
kubectl exec nginx-phppfm -c nginx-container -- cat /var/www/html/index.php
```

**Purpose**: Confirm the `index.php` file is correctly copied to the shared volume.

---

## 📚 Quick Command Reference
```bash
# Apply configurations
kubectl apply -f nginx-phpfpm-task3.yaml

# Copy index.php
kubectl cp /opt/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container

# Verify resources
kubectl get configmap nginx-config
kubectl get pod nginx-phpfpm
kubectl get svc nginx-phpfpm-service

# Verify ConfigMap mount
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/nginx.conf

# Verify shared volume
kubectl exec nginx-phpfpm -c nginx-container -- ls /var/www/html

# Test application
curl http://<node-ip>:30012
```

---

## 🛠️ Troubleshooting Common Issues

### Issue 1: Pod Not Running
**Symptoms**: Pod in `Pending` or `CrashLoopBackOff`.  
**Solution**: Check pod events and logs.
```bash
kubectl describe pod nginx-phpfpm
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container
```

### Issue 2: Service Has No Endpoints
**Symptoms**: `kubectl describe svc nginx-phpfpm-service` shows `Endpoints: <none>`.  
**Solution**: Verify selector and port alignment.
```bash
kubectl get pod -l app=nginx-phpfpm -o wide
kubectl describe svc nginx-phpfpm-service
```

### Issue 3: ConfigMap Not Mounted
**Symptoms**: `cat /etc/nginx/nginx.conf` does not show custom configuration.  
**Solution**: Verify ConfigMap and volume mount.
```bash
kubectl describe configmap nginx-config
kubectl describe pod nginx-phpfpm
```

### Issue 4: File Copy Failure
**Symptoms**: `index.php` not found in `/var/www/html`.  
**Solution**: Verify `kubectl cp` command and container name.
```bash
kubectl cp /opt/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
kubectl exec nginx-phpfpm -c nginx-container -- ls /var/www/html
```

### Issue 5: Application Not Accessible
**Symptoms**: `curl http://<node-ip>:30012` fails or does not return "It works!".  
**Solution**: Check nginx and PHP-FPM connectivity.
```bash
kubectl exec nginx-phpfpm -c nginx-container -- curl localhost:8093
```

---

## 💡 Additional Tips
- **Pod Logs**: Check for errors in nginx or PHP-FPM.
  ```bash
  kubectl logs nginx-phpfpm -c nginx-container
  kubectl logs nginx-phpfpm -c php-fpm-container
  ```
- **Dry Run**: Test configurations before applying.
  ```bash
  kubectl apply -f nginx-phpfpm-task3.yaml --dry-run=client
  ```
- **NodePort Range**: Ensure 30012 is within 30000-32767.
- **Application Health**: Verify nginx and PHP-FPM connectivity.
  ```bash
  kubectl exec nginx-phpfpm -c nginx-container -- curl localhost:8093
  ```

---

## 🚨 Task-Specific Challenge & Solution

### 🔍 Main Challenges Encountered:
1. **Nginx Configuration**: Customizing `nginx.conf` with the correct port, root, and index settings.
2. **Shared Volume**: Ensuring `emptyDir` is correctly shared between nginx and PHP-FPM.
3. **ConfigMap Mount**: Mounting `nginx.conf` with `subPath` to avoid overriding the entire directory.
4. **File Copy**: Copying `index.php` to the correct container and path.

### 💡 Solution Approach:
1. **ConfigMap**: Created `nginx-config` with custom `nginx.conf` settings.
2. **Pod**: Configured `nginx-phpfpm` with nginx and PHP-FPM containers, shared `emptyDir` volume, and ConfigMap volume.
3. **Service**: Set up `nginx-phpfpm-service` as NodePort with `nodePort: 30012`.
4. **File Copy**: Used `kubectl cp` to copy `index.php` to the nginx container.
5. **Verification**: Confirmed pod status, service accessibility, and application output.

### 🎯 Key Success Factors:
- **ConfigMap**: Correctly configures nginx with custom settings.
- **Pod**: Runs nginx and PHP-FPM with shared volume.
- **Service**: Exposes the application on the specified NodePort.
- **Application**: Displays "It works!" via the **App** button.

### ⚠️ Critical Configuration Details:
- **ConfigMap**: `nginx-config`, custom `nginx.conf`
- **Pod**: `nginx-phpfpm`, containers `nginx-container` and `php-fpm-container`
- **Volumes**: `shared-files` (emptyDir), `nginx-config-volume` (ConfigMap)
- **Service**: `nginx-phpfpm-service`, NodePort 30012
- **File**: `index.php` copied to `/var/www/html`

### 🔒 Kubernetes Benefits:
- **Flexibility**: ConfigMap allows easy nginx configuration updates.
- **Shared Storage**: `emptyDir` enables communication between containers.
- **Accessibility**: NodePort provides external access.
- **Reliability**: Kubernetes ensures pod health.

---

## ⚠️ Important Production Notes
🔧 **Pod vs. Deployment**: Consider using a Deployment for scalability and rolling updates.  
🔐 **Security**: Implement network policies to restrict access.  
📊 **Monitoring**: Monitor nginx and PHP-FPM performance.  
🛡️ **Persistence**: Replace `emptyDir` with a PersistentVolume for production.  
🌐 **Load Balancing**: Use an Ingress or LoadBalancer for production traffic.

---

## 📖 Learning Outcomes
**Key Concepts Mastered**:
- **ConfigMap**: Managing custom nginx configurations.
- **Pod**: Running multi-container applications with shared volumes.
- **Service**: Exposing applications via NodePort.
- **Volume Mounts**: Using `emptyDir` and ConfigMap volumes with `subPath`.
- **File Management**: Copying files into containers with `kubectl cp`.

**Kubernetes Features Used**:
- `v1 ConfigMap` for nginx configuration.
- `v1 Pod` for nginx and PHP-FPM containers.
- `v1 Service` for NodePort exposure.

---

## 🎯 Task Completion Summary
✅ **Successfully Completed**:
- **ConfigMap** - Created `nginx-config` with custom `nginx.conf`.
- **Pod** - Created `nginx-phpfpm` with nginx and PHP-FPM containers, shared volume, and ConfigMap mount.
- **Service** - Created `nginx-phpfpm-service` with NodePort 30012.
- **File Copy** - Copied `index.php` to `/var/www/html` in the nginx container.
- **Verification** - Confirmed pod is running, service is accessible, and application displays "It works!".

**Final Status**: 🎉 **Task completed successfully with all requirements met**

**Application Foundation**: The nginx and PHP-FPM application is deployed, accessible via NodePort 30012, and displays "It works!", ready for testing and future enhancements by the Nautilus DevOps team.

### 🔮 Application Ready for:
- **Testing**: Validate PHP application performance.
- **Scaling**: Convert to a Deployment for replicas and updates.
- **Security**: Add network policies and HTTPS.
- **Monitoring**: Implement health checks and logging.
- **Persistence**: Use a PersistentVolume for production data.