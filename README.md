# Jenkins CI/CD Pipeline with Ansible and Kubernetes(kubeadm)

## Overview

This project demonstrates a complete Jenkins-based CI/CD pipeline integrated with:
- **Kubernetes** for container orchestration
- **Ansible** for configuration management
- **Docker** for containerization
- **Persistent storage** for Jenkins data

## Architecture Components

### 1. Jenkins Infrastructure
- **Custom Jenkins Master Image** (Dockerfile)
  - Based on official Jenkins LTS
  - Includes Docker, Ansible, and SSH capabilities
  - Runs as non-root user with Docker group access

- **Kubernetes Deployment** (jenkins-deployment.yaml)
  - Single replica deployment
  - Persistent volume for Jenkins home
  - Exposed via NodePort service (30080)

### 2. Supporting Kubernetes Resources
- **Persistent Volume** (jenkins-pv.yaml)
  - 5GB hostPath volume at `/mnt/data/jenkins`
  
- **Persistent Volume Claim** (jenkins-pvc.yaml)
  - Claims the 5GB volume for Jenkins

### 3. Ansible Integration
- **Ansible Pod** (ansible-pod.yaml)
  - Runs custom Jenkins-Ansible image
  - Sleeps indefinitely to maintain connection

- **Apache Deployment Playbook** (deploy-httpd.yaml)
  - Installs and starts httpd service on worker nodes

### 4. Jenkins Pipeline (Jenkinsfile)
- Two-stage pipeline:
  1. Clone repository
  2. Execute Ansible playbook

## Deployment Instructions

### 1. Build Custom Jenkins Image
```bash
docker build -t <your-dockerhub-username>/jenkins-ansible:v1 .
docker push <your-dockerhub-username>/jenkins-ansible:v1
```

### 2. Deploy Kubernetes Resources
```bash
kubectl apply -f jenkins-pv.yaml
kubectl apply -f jenkins-pvc.yaml
kubectl apply -f jenkins-deployment.yaml
kubectl apply -f jenkins-service.yaml
kubectl apply -f ansible-pod.yaml
```

### 3. Access Jenkins
1. Get Node IP:
   ```bash
   kubectl get nodes -o wide
   ```
2. Access Jenkins at: `http://<node-ip>:30080`

### 4. Configure Jenkins Pipeline
1. Create new Pipeline job
2. Point to the Jenkinsfile in your repository
3. Add necessary credentials (SSH keys, etc.)

## Key Features

1. **Persistent Jenkins Configuration**:
   - PV/PVC ensures Jenkins data survives pod restarts
   - 5GB storage for plugins and job history

2. **Docker-in-Docker**:
   - Jenkins can build and run Docker containers
   - Jenkins user added to Docker group

3. **Ansible Integration**:
   - Dedicated Ansible pod for configuration management
   - Sample playbook for Apache deployment

4. **Kubernetes Native**:
   - Properly orchestrated deployment
   - NodePort service for external access

## Security Considerations

1. **Jenkins Security**:
   - Change default admin password
   - Setup proper user roles and permissions
   - Consider HTTPS for web UI

2. **Kubernetes Security**:
   - hostPath volumes are convenient but less secure
   - Consider using proper storage class in production
   - Implement NetworkPolicies

3. **Ansible Security**:
   - Use SSH keys instead of password authentication
   - Restrict playbook permissions

## Customization Options

1. **Storage Configuration**:
   - Modify jenkins-pv.yaml for different storage backends
   - Adjust capacity in jenkins-pvc.yaml

2. **Jenkins Image**:
   - Add additional tools to Dockerfile as needed
   - Customize plugins in derived image

3. **Ansible Playbooks**:
   - Extend deploy-httpd.yaml for additional configurations
   - Add more playbooks to the repository

## Troubleshooting

1. **Jenkins Not Accessible**:
   - Verify NodePort service is created
   - Check firewall rules on worker nodes

2. **Persistent Volume Issues**:
   - Ensure hostPath directory exists and has correct permissions
   - Verify PVC is bound to PV

3. **Ansible Connection Problems**:
   - Check ansible pod is running
   - Verify inventory.ini has correct node information

4. **Pipeline Failures**:
   - Check Jenkins logs
   - Verify repository URL in Jenkinsfile