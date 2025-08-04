# Infrastructure Automation Project

This project demonstrates a complete infrastructure setup using Ansible for automation, Kubernetes for container orchestration, and various monitoring solutions.

## Project Structure

```
├── ansible/                    # Ansible automation scripts
│   ├── roles/
│   │   ├── docker/            # Docker installation role
│   │   ├── kube/              # Kubernetes installation role
│   │   ├── kafka/             # Kafka cluster setup
│   │   └── postgres/          # PostgreSQL database setup
├── docker/                    # Docker configurations
├── kubernetes/                # Kubernetes manifests and Helm charts
│   └── simple-nginx/          # Nginx Helm chart
├── monitoring/                # Monitoring setup commands
└── kubeadm-config.yml        # Kubernetes cluster configuration
```

## Prerequisites

- Multiple Linux servers (1 control plane + 2 worker nodes)
- Ansible installed on control machine
- SSH access to all target servers

## Setup Instructions

### 1. Infrastructure Preparation with Ansible

First, configure all nodes with Docker and Kubernetes prerequisites:

```bash
# Run the Ansible playbook to install Docker and Kubernetes on all nodes
ansible-playbook -i inventory.ini playbook.yml
```

This playbook includes:
- **docker role**: Installs Docker on all nodes
- **kube role**: Installs Kubernetes components (kubelet, kubeadm, kubectl)

### 2. Kubernetes Cluster Initialization

#### Initialize the Control Plane

On the control plane node:

```bash
# Initialize the cluster with custom configuration
sudo kubeadm init --config kubeadm-config.yml
```

#### Configure kubectl Access

```bash
# Set up kubectl for the current user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Verify Initial Setup

```bash
# Check node status (should show NotReady initially)
kubectl get nodes
```

#### Install Calico Network Add-On

```bash
# Install Calico CNI plugin
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

# Verify node is now Ready
kubectl get nodes
```

#### Join Worker Nodes

Get the join command from control plane:

```bash
# Generate join command
kubeadm token create --print-join-command
```

Run the join command on each worker node:

```bash
# On worker nodes (run as root)
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

#### Verify Cluster

```bash
# Confirm all nodes are Ready
kubectl get nodes
```

### 3. Database and Messaging Setup

Deploy PostgreSQL and Kafka clusters using Ansible:

```bash
# Deploy PostgreSQL cluster with replication
ansible-playbook -i inventory.ini playbook.yml --tags postgres

# Deploy Kafka cluster
ansible-playbook -i inventory.ini playbook.yml --tags kafka
```

**PostgreSQL Features:**
- Master-slave replication setup
- Custom configuration templates
- Automated service management

**Kafka Features:**
- Multi-broker cluster
- Custom server properties
- Service management and monitoring

### 4. Docker Application

Simple Docker application setup:

```bash
# Navigate to docker directory
cd docker/

# Build the Docker image
docker build -t simple-app .

# Run the container
docker run -d -p 80:80 simple-app
```

### 5. Kubernetes Application Deployment

Deploy applications using Helm charts:

```bash
# Navigate to kubernetes directory
cd kubernetes/

# Install the Nginx application using Helm
helm install simple-nginx ./simple-nginx/

# Verify deployment
kubectl get pods
kubectl get services
```

**Features:**
- Helm chart for easy deployment
- Horizontal Pod Autoscaler (HPA) configured
- Ingress controller for external access
- Service mesh ready

#### Horizontal Pod Autoscaler

The HPA is configured to automatically scale pods based on CPU utilization:

```bash
# Check HPA status
kubectl get hpa

# Apply HPA configuration
kubectl apply -f hpa.yaml
```

### 6. Monitoring Setup

Install Prometheus for cluster monitoring:

```bash
# Navigate to monitoring directory
cd monitoring/

# Run monitoring setup commands
./commands
```

This includes:
- Prometheus server installation
- Metrics collection configuration
- Dashboard setup for cluster monitoring

## Architecture Overview

1. **Infrastructure Layer**: Automated with Ansible
   - Docker runtime on all nodes
   - Kubernetes cluster setup
   - Database and messaging services

2. **Container Orchestration**: Kubernetes
   - Multi-node cluster with Calico networking
   - Helm charts for application deployment
   - Auto-scaling capabilities

3. **Data Layer**: 
   - PostgreSQL with replication
   - Kafka for message streaming

4. **Monitoring**: Prometheus-based observability

## Key Components

- **Ansible Roles**: Modular automation for infrastructure setup
- **Kubernetes Cluster**: Production-ready container orchestration
- **Helm Charts**: Templated Kubernetes deployments
- **Auto-scaling**: HPA for dynamic resource management
- **Monitoring**: Comprehensive metrics collection

## Usage

1. Run Ansible playbooks to prepare infrastructure
2. Initialize Kubernetes cluster following the step-by-step guide
3. Deploy applications using provided Helm charts
4. Monitor the system using Prometheus dashboards

## Notes

- Ensure all nodes meet Kubernetes system requirements
- Network connectivity between all nodes is required
- Monitoring data persists according to Prometheus retention policies
- HPA requires metrics-server to be running in the cluster