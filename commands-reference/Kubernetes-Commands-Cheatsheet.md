# 🚀 Kubernetes Commands Cheatsheet

This cheatsheet provides essential `kubectl` commands for managing Kubernetes resources, based on the tasks in this repository (Levels 1-4). Commands are categorized for quick reference, with examples drawn from common scenarios like deploying pods/deployments (Level 1 Tasks 1-14), troubleshooting (Level 2 Task 10), storage/secrets (Level 3 Tasks 4-5), and multi-tier apps (Level 4 Tasks 1-5). Use `kubectl` on the jump host (`thor@jumphost`).

**Quick Tips**:
- Replace `<resource-name>` with actual names (e.g., `redis-master`).
- Use `--dry-run=client` to validate YAML without applying: `kubectl apply -f file.yaml --dry-run=client`.
- Namespace: Most tasks use `default`; specify `-n <namespace>` if needed.
- Output formats: Add `-o yaml` for full details or `-o wide` for IPs.

---

## 📋 Basic Operations

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl version` | Check client/server versions | `kubectl version --short` |
| `kubectl cluster-info` | View cluster details | `kubectl cluster-info dump` |
| `kubectl config view` | View current context | `kubectl config current-context` |
| `kubectl apply -f <file.yaml>` | Apply YAML manifest | `kubectl apply -f guestbook-task5.yaml` (Level 4 Task 5) |
| `kubectl delete -f <file.yaml>` | Delete from YAML | `kubectl delete -f nginx-deployment.yaml` (Level 1 Task 3) |
| `kubectl get all` | List all resources | `kubectl get all -l app=guestbook` (Level 4 Task 5) |

---

## 🐳 Pods

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get pods` | List pods | `kubectl get pods -l app=drupal` (Level 4 Task 4) |
| `kubectl describe pod <name>` | Describe pod details | `kubectl describe pod nginx-phpfpm` (Level 4 Task 3) |
| `kubectl logs <pod-name>` | View pod logs | `kubectl logs redis-master-xyz -c master-redis-datacenter` (Level 4 Task 1) |
| `kubectl logs <pod-name> -c <container>` | Container-specific logs | `kubectl logs frontend-abc -c php-redis-datacenter` (Level 4 Task 5) |
| `kubectl exec -it <pod-name> -- <cmd>` | Exec into pod | `kubectl exec -it mysql-pod -- mysql -u root -p` (Level 3 Task 2) |
| `kubectl port-forward <pod> <local>:<pod-port>` | Forward port | `kubectl port-forward drupal-pod 8080:80` (Level 4 Task 4) |

---

## 📦 Deployments & ReplicaSets

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get deployments` | List deployments | `kubectl get deployments -l tier=backend` (Level 4 Task 5) |
| `kubectl describe deployment <name>` | Describe deployment | `kubectl describe deployment redis-slave` (Level 4 Task 1) |
| `kubectl scale deployment <name> --replicas=<n>` | Scale replicas | `kubectl scale deployment frontend --replicas=3` (Level 4 Task 5) |
| `kubectl rollout status deployment <name>` | Check rollout status | `kubectl rollout status deployment nginx` (Level 1 Task 3) |
| `kubectl rollout history deployment <name>` | View rollout history | `kubectl rollout history deployment httpd` (Level 1 Task 5) |
| `kubectl rollout undo deployment <name>` | Rollback deployment | `kubectl rollout undo deployment httpd` (Level 1 Task 5) |
| `kubectl set image deployment <name> <container>=<image>` | Update image | `kubectl set image deployment nginx nginx=nginx:1.21` (Level 1 Task 12) |

---

## 🌐 Services & Ingress

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get services` | List services | `kubectl get svc -l app=guestbook` (Level 4 Task 5) |
| `kubectl describe service <name>` | Describe service | `kubectl describe svc frontend` (Level 4 Task 5) |
| `kubectl expose deployment <name> --type=NodePort --port=80` | Expose as NodePort | `kubectl expose deployment drupal --type=NodePort --port=80 --name=drupal-service` (Level 4 Task 4) |
| `kubectl port-forward service <name> <local>:<port>` | Forward service port | `kubectl port-forward svc mysql 3306:3306` (Level 3 Task 2) |
| `kubectl get endpoints <service>` | View service endpoints | `kubectl get endpoints nginx-phpfpm-service` (Level 4 Task 3) |

---

## 💾 Volumes, ConfigMaps, & Secrets

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get pv` | List PersistentVolumes | `kubectl get pv drupal-mysql-pv` (Level 4 Task 4) |
| `kubectl get pvc` | List PersistentVolumeClaims | `kubectl get pvc mysql-pv-claim` (Level 4 Task 2) |
| `kubectl get configmap` | List ConfigMaps | `kubectl get configmap nginx-config` (Level 4 Task 3) |
| `kubectl get secret` | List Secrets | `kubectl get secret mysql-root-pass` (Level 4 Task 2) |
| `kubectl create configmap <name> --from-file=<file>` | Create ConfigMap from file | `kubectl create configmap nginx-config --from-file=nginx.conf` (Level 4 Task 3) |
| `kubectl create secret generic <name> --from-literal=<key>=<value>` | Create Secret | `kubectl create secret generic mysql-secret --from-literal=password=secret123` (Level 3 Task 5) |
| `kubectl describe pvc <name>` | Describe PVC | `kubectl describe pvc drupal-mysql-pvc` (Level 4 Task 4) |

---

## 🔍 Labels & Selectors

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get pods -l <label>=<value>` | List by label | `kubectl get pods -l app=guestbook,tier=frontend` (Level 4 Task 5) |
| `kubectl label pod <name> <key>=<value>` | Add label to pod | `kubectl label pod nginx-pod app=webserver` (Level 1 Task 3) |
| `kubectl annotate pod <name> <key>=<value>` | Add annotation | `kubectl annotate pod mysql-pod description="LAMP DB"` (Level 3 Task 2) |

---

## 🛠️ Troubleshooting & Debugging

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl get events --sort-by=.metadata.creationTimestamp` | List recent events | `kubectl get events --field-selector involvedObject.name=redis-pod` (Level 4 Task 1) |
| `kubectl debug <pod> -it --image=busybox` | Debug pod | `kubectl debug frontend-pod -it --image=busybox` (Level 4 Task 5) |
| `kubectl top pods` | View pod resource usage | `kubectl top pods -l app=drupal` (Level 4 Task 4) |
| `kubectl cp <local> <pod>:/path -c <container>` | Copy files to/from pod | `kubectl cp index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container` (Level 4 Task 3) |
| `kubectl rollout restart deployment <name>` | Restart deployment | `kubectl rollout restart deployment mysql` (Level 3 Task 8) |

---

## 📊 Advanced & Utils

| Command | Description | Example |
|---------|-------------|---------|
| `kubectl apply -k <dir>` | Apply Kustomize | `kubectl apply -k overlays/prod` (for future advanced tasks) |
| `kubectl wait --for=condition=available deployment/<name>` | Wait for readiness | `kubectl wait --for=condition=available deployment/frontend --timeout=60s` (Level 4 Task 5) |
| `kubectl edit deployment <name>` | Edit live resource | `kubectl edit deployment nginx` (Level 1 Task 12) |
| `kubectl patch deployment <name> -p '{"spec":{"replicas":2}}'` | Patch resource | `kubectl patch deployment redis-slave -p '{"spec":{"replicas":2}}'` (Level 4 Task 1) |
| `kubectl auth can-i <verb> <resource>` | Check permissions | `kubectl auth can-i get pods` |

**Pro Tips from Repo Tasks**:
- For NodePort services (common in Levels 1-4): Always verify with `kubectl get svc -o wide` and test via `curl http://<node-ip>:<node-port>`.
- In troubleshooting tasks (Level 2 Task 10, Level 3 Task 8): Start with `kubectl describe` and `kubectl logs`.
- For multi-container pods (Level 4 Task 3): Use `-c <container>` for logs/exec.

For more, explore the task files in `level-1/` to `level-4/`. Happy kubectl-ing! 🚀

*Last Updated: October 04, 2025*