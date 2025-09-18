# 🌟 KodeKloud Kubernetes Solutions

**📌 Project Overview**

This repository contains solutions for Kubernetes lab tasks from KodeKloud, organized by difficulty levels (Level 1 to Level 4). Each task is documented in a detailed Markdown file with step-by-step instructions, YAML manifests, verification commands, troubleshooting tips, and production notes. These solutions are designed to assist the Nautilus DevOps team in deploying and managing Kubernetes resources efficiently. It is important to note that when starting any Kubernetes task, users are already logged into the Jump Host by default as the thor user. Therefore, the SSH command to connect again is for educational purpose.  This repository serves as a comprehensive record of my learning experience through the KodeKloud Engineer challenge, showcasing real-world tasks and projects designed to build practical DevOps kubernetes  expertise. The tasks documented here reflect the specific challenges I encountered during the challenge, including detailed solutions and verification steps. Note: While the core objectives and challenges remain consistent, the specific values (e.g., server names, file paths, or other parameters) in these tasks may differ from those you encounter in your own environment. However, the underlying concepts and problem-solving approaches remain applicable, enabling you to adapt the solutions to your specific context

**🔍 Purpose**: Provide clear, executable solutions for Kubernetes labs, ensuring best practices and alignment with task requirements.

**🛠️ Tools Used**:
- **Kubernetes**: Managed via `kubectl` on the jump host.
- **Images**: Official images (e.g., `nginx:latest`, `debian:latest`, `jenkins/jenkins`).
- **Environment**: KodeKloud lab environment with pre-configured `kubectl`.

**📂 Repository Structure**:
```
kodekloud-kubernetes-solutions/
├── level-1/
│   ├── Task-01-VolumeShareDevops.md
│   ├── Task-02-WebserverPod.md
│   ├── Task-03-NginxDeployment.md
│   ├── Task-04-PodResourceLimits.md
│   ├── Task-05-HttpdDeploymentRollback.md
│   ├── Task-06-JenkinsDeployment.md
│   ├── Task-07-GrafanaDeployment.md
│   ├── Task-08-TomcatDeployment.md
│   ├── Task-09-NodeDeployment.md
│   ├── Task-10-TimeCheckPod.md
│   ├── Task-11-SidecarFix.md
│   ├── Task-12-UpdateDeploymentService.md
│   ├── Task-13-ReplicationController.md
│   ├── Task-14-NginxPHPFPMFix.md
├── level-2/
│   ├── Task-01-VolumeShareDevops.md
│   ├── Task-02-WebserverPod.md
│   ├── Task-03-NginxDeployment.md
│   ├── Task-04-PrintEnvarsGreeting.md
│   ├── Task-05-HttpdDeploymentRollback.md
│   ├── Task-06-JenkinsDeployment.md
│   ├── Task-07-GrafanaDeployment.md
│   ├── Task-08-TomcatDeployment.md
│   ├── Task-09-NodeDeployment.md
│   ├── Task-10-TBD.md
│   ├── Task-11-TBD.md
├── level-3/
│   └── *Coming Soon*
├── level-4/
│   └── *Coming Soon*
├── commands-reference/
│   └── Kubernetes-Commands-Cheatsheet.md (*TBD*)
├── troubleshooting/
│   └── Common-Issues-Solutions.md (*TBD*)
└── README.md
```

---

## 🚀 Level 1 Tasks

| Task # | Title/Link | Status | Difficulty | Description |
|--------|------------|--------|------------|-------------|
| 1 | [**Volume Share Devops**](./level-1/Task-01-VolumeShareDevops.md) | ✅ **Done** | 🟢 Basic | Create a pod with shared volume between two containers |
| 2 | [**Webserver Pod**](./level-1/Task-02-WebserverPod.md) | ✅ **Done** | 🟢 Basic | Create a pod with nginx and sidecar logging container |
| 3 | [**Nginx Deployment**](./level-1/Task-03-NginxDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy nginx with 3 replicas and NodePort service |
| 4 | [**Pod with Resource Limits**](./level-1/Task-04-PodResourceLimits.md) | ✅ **Done** | 🟡 Intermediate | Set CPU and memory limits for an httpd pod |
| 5 | [**Httpd Deployment Rollback**](./level-1/Task-05-HttpdDeploymentRollback.md) | ✅ **Done** | 🟡 Intermediate | Deploy httpd, update, and rollback with NodePort service |
| 6 | [**Jenkins Deployment**](./level-1/Task-06-JenkinsDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Jenkins with NodePort service |
| 7 | [**Grafana Deployment**](./level-1/Task-07-GrafanaDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Grafana with NodePort service |
| 8 | [**Tomcat Deployment**](./level-1/Task-08-TomcatDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Tomcat with NodePort service |
| 9 | [**Node Deployment**](./level-1/Task-09-NodeDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Node app with 2 replicas and NodePort service |
| 10 | [**Time Check Pod**](./level-1/Task-10-TimeCheckPod.md) | ✅ **Done** | 🟢 Basic | Create a pod to display time every 10 seconds |
| 11 | [**Sidecar Fix**](./level-1/Task-11-SidecarFix.md) | ✅ **Done** | 🟡 Intermediate | Fix a sidecar container logging issue |
| 12 | [**Update Deployment Service**](./level-1/Task-12-UpdateDeploymentService.md) | ✅ **Done** | 🟡 Intermediate | Update deployment and service for an application |
| 13 | [**Replication Controller**](./level-1/Task-13-ReplicationController.md) | ✅ **Done** | 🟡 Intermediate | Create a replication controller for an application |
| 14 | [**Nginx PHP-FPM Fix**](./level-1/Task-14-NginxPHPFPMFix.md) | ✅ **Done** | 🟡 Intermediate | Fix an Nginx and PHP-FPM application deployment |

---

## 🚀 Level 2 Tasks

| Task # | Title/Link | Status | Difficulty | Description |
|--------|------------|--------|------------|-------------|
| 1 | [**Volume Share Devops**](./level-2/Task-01-VolumeShareDevops.md) | ✅ **Done** | 🟢 Basic | Create a pod with shared volume between two containers |
| 2 | [**Webserver Pod**](./level-2/Task-02-WebserverPod.md) | ✅ **Done** | 🟢 Basic | Create a pod with nginx and sidecar logging container |
| 3 | [**Nginx Deployment**](./level-2/Task-03-NginxDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy nginx with 3 replicas and NodePort service |
| 4 | [**Print Envars Greeting**](./level-2/Task-04-PrintEnvarsGreeting.md) | ✅ **Done** | 🟢 Basic | Create a pod with environment variables and echo command |
| 5 | [**Httpd Deployment Rollback**](./level-2/Task-05-HttpdDeploymentRollback.md) | ✅ **Done** | 🟡 Intermediate | Deploy httpd, update, and rollback with NodePort service |
| 6 | [**Jenkins Deployment**](./level-2/Task-06-JenkinsDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Jenkins with NodePort service |
| 7 | [**Grafana Deployment**](./level-2/Task-07-GrafanaDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Grafana with NodePort service |
| 8 | [**Tomcat Deployment**](./level-2/Task-08-TomcatDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Tomcat with NodePort service |
| 9 | [**Node Deployment**](./level-2/Task-09-NodeDeployment.md) | ✅ **Done** | 🟡 Intermediate | Deploy Node app with 2 replicas and NodePort service |
| 10 | [**TBD**](./level-2/Task-10-TBD.md) | ⏳ **In Progress** | 🟡 Intermediate | Placeholder for Level 2 Task 10 |
| 11 | [**TBD**](./level-2/Task-11-TBD.md) | ⏳ **In Progress** | 🟡 Intermediate | Placeholder for Level 2 Task 11 |

---

## 🚧 Level 3 Tasks (*Coming Soon*)

Level 3 tasks are in development and will cover advanced Kubernetes concepts such as ConfigMaps, Secrets, StatefulSets, and RBAC. Stay tuned for updates!

---

## 🚧 Level 4 Tasks (*Coming Soon*)

Level 4 tasks are planned and will include expert-level Kubernetes topics such as cluster administration, network policies, and advanced storage solutions. Check back for updates!

---

## 💡 How to Use This Repository

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/MiqdadProjects/kodekloud-kubernetes-solutions.git
   cd kodekloud-kubernetes-solutions
   ```

2. **Navigate to Task Files**:
   - Level 1 tasks: `level-1/`
   - Level 2 tasks: `level-2/`
   - Future levels: `level-3/` and `level-4/`

3. **Apply Solutions**:
   - Copy the YAML manifests from each task’s Markdown file.
   - Run `kubectl apply -f <file>.yaml` to deploy resources.
   - Follow verification steps (e.g., `kubectl get pods`, `kubectl logs`) to ensure success.

4. **Troubleshooting**:
   - Refer to the "Troubleshooting Common Issues" section in each task file.
   - Use commands like `kubectl describe pod` or `kubectl logs` for debugging.
   - Check the `troubleshooting/Common-Issues-Solutions.md` file (TBD) for common fixes.

5. **Test in Lab Environment**:
   - Use the jump host (`ssh tony@jumphost`) with pre-configured `kubectl`.
   - Verify application accessibility via lab buttons (e.g., "App," "Jenkins," "Grafana").

---

## 🔧 Prerequisites

- **Kubernetes Cluster**: Access to a Kubernetes cluster via `kubectl` on the jump host.
- **Text Editor**: VS Code or similar for editing Markdown and YAML files.
- **Git**: For version control and repository management.
- **Lab Environment**: KodeKloud lab with pre-configured jump host.

---

## 📋 Detailed Setup Guide

1. **Access the Jump Host**:
   ```bash
   ssh tony@jumphost
   ```

2. **Create and Edit YAML Files**:
   ```bash
   vi <task-file>.yaml
   # Paste YAML content from the task’s Markdown file
   # Save with :wq
   ```

3. **Apply Manifests**:
   ```bash
   kubectl apply -f <task-file>.yaml
   ```

4. **Verify Resources**:
   ```bash
   kubectl get pods
   kubectl get svc
   kubectl get deployment
   kubectl logs <pod-name>
   curl http://<node-ip>:<node-port>
   ```

5. **Test Application Access**:
   - Use `curl` or the lab’s UI buttons (e.g., "App," "NodeApp") to verify accessibility.
   - Example: For Task 3, click the "App" button to check nginx on port 30011.

6. **Debug Issues**:
   - Check pod status: `kubectl describe pod <pod-name>`
   - View logs: `kubectl logs <pod-name> -c <container-name>`
   - Verify services: `kubectl describe svc <service-name>`

---

## ⚠️ Important Notes

- **Filenames**: Task files use the format `Task-XX-DescriptiveName.md` to avoid VS Code naming issues (e.g., resolved issue with `Task-04-Pod-Resource-Limits.md` by using `Task-04-PodResourceLimits.md`).
- **Image Tags**: Always use specified image tags (e.g., `nginx:latest`, `httpd:2.4.27`) to meet task requirements.
- **Dry Runs**: Test manifests with `kubectl apply -f <file>.yaml --dry-run=client` to validate syntax.
- **Production Readiness**: Each task includes production notes for real-world considerations.
- **Level 2 Completion**: Tasks 10 and 11 are placeholders (`Task-10-TBD.md`, `Task-11-TBD.md`) pending task details.

---

## 🚀 Contributing

Contributions are welcome! To add new tasks, improve solutions, or update documentation:
1. Fork the repository.
2. Create a new branch:
   ```bash
   git checkout -b task-XX
   ```
3. Add or update Markdown/YAML files in the appropriate level directory.
4. Commit changes:
   ```bash
   git commit -m "Add Task XX solution"
   ```
5. Push to the branch:
   ```bash
   git push origin task-XX
   ```
6. Open a pull request with a detailed description of changes.

---

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.

---

## 📞 Contact

For questions, issues, or task details (e.g., Level 2 Tasks 10 and 11), contact the Nautilus DevOps team or open an issue in the repository.


**Happy Kuberneting!** 🚀


