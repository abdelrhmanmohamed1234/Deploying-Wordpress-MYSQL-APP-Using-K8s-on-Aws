# 🚀 Deploying WordPress & MySQL on a Self-Managed Kubernetes Cluster (AWS)

> A production-grade deployment of a highly available WordPress site backed by a MySQL database, running on a self-managed Kubernetes cluster built with **kubeadm** on **AWS EC2 instances**.

---

## 📌 Project Overview

This project demonstrates a complete cloud-native infrastructure setup using Kubernetes — without relying on managed services like EKS. The cluster is bootstrapped from scratch using **kubeadm** across **3 AWS EC2 instances**, with full integration of AWS storage services (EBS & EFS).

**Project designed by:** Eng. Abdelrhman Mohmaed | abd0001102002@gmail.com)

---

## 🏗️ Architecture

```
Internet / Users
       │
       ▼
AWS Load Balancer (High Availability)
       │
       ▼
┌─────────────────────────────────────────────────┐
│           Kubernetes Cluster (Self-Managed)      │
│                                                  │
│  ┌──────────────────┐   ┌─────────────────────┐  │
│  │  WordPress Pod   │   │    MySQL Pod        │  │
│  │  - PVC (EFS)     │   │  - PVC (EBS)        │  │
│  │  - Env Vars      │◄──│  - Secret           │  │
│  │  - LB Service    │   │  - ClusterIP Svc    │  │
│  └──────────────────┘   └─────────────────────┘  │
│                                                  │
│  Node 1 (Control Plane) | Node 2 | Node 3        │
└─────────────────────────────────────────────────┘
```

---

## ⚙️ Cluster Setup

| Component        | Details                          |
|------------------|----------------------------------|
| Cloud Provider   | AWS                              |
| Instances        | 3 × EC2 (t3.medium recommended)  |
| Node Roles       | 1 Control Plane + 2 Worker Nodes |
| Bootstrap Tool   | kubeadm                          |
| CNI Plugin       | Flannel / Calico                 |

---

## 📦 Project Structure

```
├── mysql/
│   ├── mysql-secret.yaml         # Kubernetes Secret for DB credentials
│   ├── mysql-storageclass.yaml   # Storage Class definition
│   ├── mysql-pvc.yaml            # Persistent Volume Claim
│   ├── mysql-deployment.yaml     # MySQL Deployment
│   └── mysql-service.yaml        # ClusterIP Service
│
├── wordpress/
│   ├── wordpress-storageclass.yaml  # Storage Class for WordPress
│   ├── wordpress-pvc.yaml           # Persistent Volume Claim
│   ├── wordpress-deployment.yaml    # WordPress Deployment
│   └── wordpress-service.yaml       # Load Balancer Service
│
└── README.md
```

---

## 🗄️ MySQL Deployment Steps

1. **Create Kubernetes Secret** — stores MySQL root password securely
2. **Define Storage Class** — enables dynamic volume provisioning
3. **Create PVC** — requests persistent storage for MySQL data
4. **Deploy MySQL** — Kubernetes Deployment with resource configuration
5. **Auto-provision PV** — automatically created when PVC is bound
6. **AWS EBS Integration** — PV backed by Elastic Block Store for durable storage
7. **ClusterIP Service** — exposes MySQL internally within the cluster only

---

## 🌐 WordPress Deployment Steps

1. **Define Storage Class** — dedicated storage class for WordPress
2. **Create PVC** — requests persistent storage for WordPress files
3. **Auto-provision PV** — automatically created when PVC is bound
4. **Deploy WordPress** — Kubernetes Deployment configured with:
   - PVC binding for file persistence
   - MySQL password via environment variable
   - MySQL service name for database connectivity
5. **AWS EFS Integration** — PV backed by Elastic File System for scalable shared storage
6. **Load Balancer Service** — exposes WordPress externally to the internet
7. **AWS Load Balancer** — configured for high availability across worker nodes

---

## 🔧 Prerequisites

- AWS account with EC2, EBS, and EFS access
- 3 EC2 instances (Ubuntu 22.04 recommended)
- `kubectl` installed on your local machine
- `kubeadm`, `kubelet`, `kubectl` installed on all EC2 nodes
- AWS EBS CSI Driver configured on the cluster
- AWS EFS CSI Driver configured on the cluster

---

## 🚀 Deployment Guide

### Step 1 — Bootstrap the Kubernetes Cluster

**On the Control Plane node:**
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Install CNI (Flannel):**
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**Join Worker Nodes** (use the token from kubeadm init output):
```bash
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

---

### Step 2 — Deploy MySQL

```bash
kubectl apply -f mysql/mysql-secret.yaml
kubectl apply -f mysql/mysql-storageclass.yaml
kubectl apply -f mysql/mysql-pvc.yaml
kubectl apply -f mysql/mysql-deployment.yaml
kubectl apply -f mysql/mysql-service.yaml
```

---

### Step 3 — Deploy WordPress

```bash
kubectl apply -f wordpress/wordpress-storageclass.yaml
kubectl apply -f wordpress/wordpress-pvc.yaml
kubectl apply -f wordpress/wordpress-deployment.yaml
kubectl apply -f wordpress/wordpress-service.yaml
```

---

### Step 4 — Access WordPress

```bash
# Get the external Load Balancer URL
kubectl get svc wordpress-service
```

Copy the **EXTERNAL-IP** and open it in your browser. 🎉

---

## 🔍 Verify Deployment

```bash
# Check all pods are running
kubectl get pods

# Check persistent volumes
kubectl get pv
kubectl get pvc

# Check services
kubectl get svc

# Check nodes
kubectl get nodes
```

---

## 🔐 Security Notes

- MySQL credentials are stored in a **Kubernetes Secret** (base64 encoded)
- MySQL is only accessible **internally** via ClusterIP — never exposed to the internet
- WordPress communicates with MySQL using the **service name** as the hostname
- It is recommended to enable **encryption at rest** for EBS volumes in production

---

## 💡 Key Takeaways

- Building a self-managed cluster with **kubeadm** deepens understanding of how Kubernetes works under the hood
- **Separating MySQL and WordPress** into different deployments improves maintainability and security
- Using **EBS for MySQL** provides durable, block-level storage ideal for databases
- Using **EFS for WordPress** provides scalable shared storage ideal for web application files
- A **Load Balancer Service** combined with AWS ensures true high availability

---

## 📎 Tags

`#Kubernetes` `#AWS` `#DevOps` `#CloudComputing` `#kubeadm` `#EC2` `#WordPress` `#MySQL` `#EBS` `#EFS` `#CloudNative` `#DolfinED`

---

## 👨‍💻 Author

**Eng. Abdelrhman Mohamed**
🌐 [abd0001102002@gmail.com)

---

> ⭐ If you found this project helpful, please give it a star on GitHub!
