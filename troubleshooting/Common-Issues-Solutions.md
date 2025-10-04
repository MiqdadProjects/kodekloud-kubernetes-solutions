# 🛠️ Common Kubernetes Issues & Solutions

This guide addresses frequent issues encountered in the KodeKloud Kubernetes tasks across Levels 1-4, based on patterns like pod crashes (Level 1 Task 11), deployment troubleshooting (Level 2 Task 10), storage binding failures (Level 3 Task 4), and service endpoint issues (Level 4 Task 3). Each issue includes symptoms, causes, and step-by-step solutions with commands. Use this alongside task-specific troubleshooting sections.

**Quick Debug Flow**:
1. `kubectl get events --sort-by=.metadata.creationTimestamp` (recent events).
2. `kubectl describe <resource> <name>` (details).
3. `kubectl logs <pod> -c <container>` (logs).
4. Test connectivity: `kubectl exec <pod> -- curl <service-ip>:<port>`.

---

## 🐳 Pod Issues

### Issue 1: Pod Stuck in Pending/CrashLoopBackOff
**Symptoms**: `kubectl get pods` shows `Pending` or `CrashLoopBackOff`; events mention scheduling/resource errors.
**Common Causes**: Insufficient resources, image pull failure, or init container issues (Level 3 Task 3).
**Solutions**:
1. Check events: `kubectl get events --field-selector involvedObject.name=<pod-name>`.
2. Describe pod: `kubectl describe pod <pod-name>` (look for "FailedScheduling" or "ErrImagePull").
3. Fix image pull: Ensure correct tag (e.g., `mysql:5.7` in Level 4 Task 2); retry: `kubectl delete pod <pod-name>`.
4. Increase resources: Edit deployment: `kubectl edit deployment <name>` and add `resources.limits.cpu: 500m`.
5. For init containers: Verify init image pulls: `kubectl logs <pod> -c init-container`.

**Example (Level 1 Task 11 Sidecar Fix)**: If sidecar crashes, check logs: `kubectl logs <pod> -c sidecar`; ensure volume mount paths match.

### Issue 2: Pod Not Ready (0/1 Ready)
**Symptoms**: `kubectl get pods` shows `0/1` ready; describe shows liveness/readiness probe failures.
**Common Causes**: Probe misconfiguration or app startup delay (Level 1 Task 4 Resource Limits).
**Solutions**:
1. Check probes: `kubectl describe pod <pod-name>` (under "Conditions").
2. Adjust probes: `kubectl edit deployment <name>`; set `initialDelaySeconds: 30` for startup.
3. View app logs: `kubectl logs <pod> --previous` (previous crash logs).
4. Restart: `kubectl rollout restart deployment <name>`.

**Example (Level 3 Task 6 Env Vars)**: If env vars cause startup failure, verify: `kubectl exec <pod> -- env | grep <var>`.

---

## 📦 Deployment Issues

### Issue 3: Deployment Not Scaling/Replicas Mismatch
**Symptoms**: `kubectl get deployments` shows wrong `READY` count; rollout stuck.
**Common Causes**: Selector mismatch or image update issues (Level 1 Task 5 Rollback).
**Solutions**:
1. Check status: `kubectl rollout status deployment <name>`.
2. View history: `kubectl rollout history deployment <name>`.
3. Rollback: `kubectl rollout undo deployment <name>`.
4. Scale: `kubectl scale deployment <name> --replicas=3` (e.g., frontend in Level 4 Task 5).
5. Fix selectors: Ensure labels match in YAML: `app: guestbook, tier: frontend`.

**Example (Level 2 Task 10 Troubleshoot)**: If replicas=0, patch: `kubectl patch deployment <name> -p '{"spec":{"replicas":1}}'`.

### Issue 4: Image PullBackOff or ErrImagePull
**Symptoms**: Pod events show "Failed to pull image"; common in custom images (Level 4 Task 1 Redis).
**Solutions**:
1. Describe: `kubectl describe pod <pod-name>` (image details).
2. Verify tag: Use exact image (e.g., `gcr.io/google_samples/gb-redisslave:v3` in Level 4 Task 5).
3. Pull manually: `kubectl delete pod <pod-name>` to retry.
4. Check registry access: Ensure no auth issues (add ImagePullSecrets if needed).

**Example (Level 3 Task 9 Iron Gallery)**: For private images, create secret: `kubectl create secret docker-registry regcred --docker-server=<server> --docker-username=<user>`.

---

## 🌐 Service Issues

### Issue 5: Service Has No Endpoints (<none>)
**Symptoms**: `kubectl describe svc <name>` shows "Endpoints: <none>"; traffic not routed.
**Common Causes**: Label selector mismatch (Level 1 Task 12 Update Service).
**Solutions**:
1. Verify selector: `kubectl describe svc <name>` vs. pod labels: `kubectl get pods -l <selector>`.
2. Fix labels: `kubectl label pod <pod> app=web --overwrite`.
3. Check ports: Ensure `targetPort` matches containerPort (e.g., 80 for frontend in Level 4 Task 5).
4. Test endpoints: `kubectl get endpoints <svc-name>`.

**Example (Level 4 Task 3 Nginx PHP-FPM)**: If no endpoints, align `app: nginx-phpfpm` in pod and service selectors.

### Issue 6: NodePort Not Accessible
**Symptoms**: `curl http://<node-ip>:<nodeport>` fails; service exists but no response.
**Common Causes**: Wrong NodePort range or firewall (Levels 1-4 NodePort tasks).
**Solutions**:
1. Get NodePort: `kubectl get svc <name> -o jsonpath='{.spec.ports[0].nodePort}'`.
2. Verify range: NodePorts 30000-32767 (e.g., 30009 in Level 4 Task 5).
3. Test locally: `kubectl port-forward svc <name> 8080:80`.
4. Check pod readiness: `kubectl get pods -l <selector>` (must be Running/Ready).

**Example (Level 1 Task 3 Nginx)**: If 30011 not working, expose again: `kubectl expose deployment nginx --type=NodePort --port=80 --name=nginx-svc`.

---

## 💾 Storage & Secrets Issues

### Issue 7: PVC Pending/Not Bound
**Symptoms**: `kubectl get pvc` shows "Pending"; describe mentions no PV available.
**Common Causes**: Storage mismatch (Level 3 Task 4 Persistent Volumes).
**Solutions**:
1. Check PV: `kubectl get pv` (ensure capacity/access modes match, e.g., 5Gi RWO in Level 4 Task 4).
2. Describe PVC: `kubectl describe pvc <name>` (look for "waiting for PV").
3. Create matching PV: Ensure `hostPath` exists (e.g., `/drupal-mysql-data` in Level 4 Task 4).
4. Delete/recreate: `kubectl delete pvc <name>; kubectl apply -f pvc.yaml`.

**Example (Level 4 Task 2 MySQL)**: If 250Mi not bound, adjust PV capacity: `kubectl patch pv <pv-name> -p '{"spec":{"capacity":{"storage":"250Mi"}}}''`.

### Issue 8: Secret Not Mounted/Env Var Missing
**Symptoms**: App logs show auth errors; `kubectl exec <pod> -- env` missing vars.
**Common Causes**: Wrong secretKeyRef (Level 3 Task 5 Manage Secrets).
**Solutions**:
1. Verify secret: `kubectl get secret <name> -o yaml` (check keys/values).
2. Check mount: `kubectl describe pod <pod-name>` (under "Volumes").
3. Fix env: Ensure `valueFrom.secretKeyRef.name=<secret>,key=<key>` in YAML.
4. Restart: `kubectl rollout restart deployment <name>`.

**Example (Level 4 Task 4 Drupal)**: If DB password missing, recreate secret: `kubectl create secret generic drupal-secret --from-literal=mysql-password=drupalpass`.

---

## 🔍 General Troubleshooting

### Issue 9: Resource Quota Exceeded
**Symptoms**: Events show "Insufficient cpu/memory"; common in resource-limited labs (Level 1 Task 4).
**Solutions**:
1. Check usage: `kubectl top nodes; kubectl top pods`.
2. Reduce requests: Edit deployment: `kubectl edit deployment <name>` (set `requests.cpu: 50m`).
3. Delete unused: `kubectl delete pod --all -l app=old-app`.

### Issue 10: Network Policy Blocking Traffic
**Symptoms**: Pods can't reach services; `kubectl exec <pod> -- curl <svc>` times out (advanced, implied in Level 3 Task 7 LEMP).
**Solutions**:
1. Check policies: `kubectl get networkpolicy`.
2. Allow traffic: Create policy YAML allowing `app: frontend` to `app: backend`.
3. Test: `kubectl exec <pod> -- nslookup <svc-name>` (DNS resolution).

**Pro Tips from Repo**:
- For LAMP/LEMP fixes (Level 2 Task 11, Level 3 Task 7): Always check port forwards and env vars.
- Multi-tier apps (Level 4): Verify DNS with `GET_HOSTS_FROM=dns` env var.
- General: Use `kubectl proxy` for API access: `kubectl proxy --port=8080`.

*Last Updated: October 04, 2025*