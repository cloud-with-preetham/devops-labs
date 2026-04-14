# Amazon EKS (Days 81–83)

This directory contains my hands-on Amazon Elastic Kubernetes Service (EKS) learning and implementation completed as part of my **90 Days of DevOps** journey.

Over these three days, I progressed from provisioning an EKS cluster with Terraform to deploying production-ready applications using Helm, integrating AWS Load Balancer Controller, configuring TLS with cert-manager, and exposing applications through Envoy Gateway.

---

## Learning Objectives

- Understand Amazon EKS architecture
- Provision infrastructure using Terraform
- Configure kubectl access to EKS
- Deploy workloads using Helm
- Install AWS Load Balancer Controller
- Manage TLS certificates using cert-manager
- Configure Gateway API with Envoy Gateway
- Expose applications securely over HTTPS

---

# Project Structure

```text
Amazon-EKS/
├── day-81/
│   ├── terraform/
│   ├── README.md
│   └── screenshots/
│
├── day-82/
│   ├── helm/
│   ├── aws-load-balancer-controller/
│   ├── README.md
│   └── screenshots/
│
├── day-83/
│   ├── envoy-gateway/
│   ├── gateway-api/
│   ├── cert-manager/
│   ├── application-deployment/
│   ├── README.md
│   └── screenshots/
│
└── README.md
```

---

# Day 81 — Provision Amazon EKS using Terraform

## Topics Covered

- AWS VPC
- IAM Roles
- Amazon EKS
- Managed Node Groups
- Terraform Modules
- kubectl Configuration

### Tasks Performed

- Created VPC
- Created EKS Cluster
- Created Managed Node Group
- Configured kubectl
- Verified worker nodes

### Tools Used

- Terraform
- AWS CLI
- kubectl
- Amazon EKS

---

# Day 82 — Deploy Applications using Helm

## Topics Covered

- Helm Charts
- Kubernetes Packages
- Helm Release Management
- AWS Load Balancer Controller

### Tasks Performed

- Installed Helm
- Created Helm charts
- Installed AWS Load Balancer Controller
- Deployed sample application
- Managed Helm releases

### Tools Used

- Helm
- Kubernetes
- Amazon EKS
- AWS IAM

---

# Day 83 — Secure Kubernetes Applications

## Topics Covered

- Gateway API
- Envoy Gateway
- cert-manager
- Let's Encrypt
- HTTPS
- DNS
- Load Balancer

### Tasks Performed

- Installed Envoy Gateway
- Installed Gateway API CRDs
- Configured Gateway
- Created HTTPRoute
- Installed cert-manager
- Created ClusterIssuer
- Generated TLS certificates
- Enabled HTTPS access

### Tools Used

- Envoy Gateway
- cert-manager
- Gateway API
- Helm
- Kubernetes
- Amazon EKS

---

# Skills Learned

- Amazon EKS
- Infrastructure as Code
- Terraform
- Kubernetes Networking
- Helm
- IAM Roles
- AWS Load Balancer Controller
- cert-manager
- Gateway API
- Envoy Gateway
- TLS Certificates
- HTTPS Configuration
- Production Kubernetes Deployment

---

# Commands Practiced

```bash
terraform init

terraform plan

terraform apply

aws eks update-kubeconfig

kubectl get nodes

helm install

helm upgrade

helm list

kubectl apply -f

kubectl get pods

kubectl get svc

kubectl describe gateway

kubectl get httproute

kubectl get certificate

kubectl logs
```

---

# Key Takeaways

- Learned how Amazon EKS simplifies Kubernetes management.
- Automated infrastructure provisioning using Terraform.
- Managed Kubernetes applications with Helm.
- Configured production-grade ingress using Envoy Gateway.
- Secured applications using cert-manager and Let's Encrypt.
- Understood Kubernetes Gateway API architecture.
- Improved AWS networking and Kubernetes deployment skills.

---

# Technologies Used

- Amazon Web Services (AWS)
- Amazon EKS
- EC2
- IAM
- VPC
- Terraform
- Kubernetes
- Helm
- Envoy Gateway
- Gateway API
- cert-manager
- Let's Encrypt
- kubectl
- AWS CLI

---

# Outcome

By completing Days **81–83**, I successfully built a production-style Kubernetes environment on Amazon EKS. The infrastructure was provisioned using Terraform, applications were deployed with Helm, networking was managed through Envoy Gateway, and secure HTTPS access was enabled using cert-manager and Let's Encrypt.

This hands-on experience strengthened my understanding of Kubernetes on AWS and real-world cloud-native deployment practices.

---

## Screenshots

Store screenshots in:

```
Amazon-EKS/
├── day-81/screenshots/
├── day-82/screenshots/
└── day-83/screenshots/
```

Recommended screenshots:

- Terraform Apply Success
- EKS Cluster Active
- Worker Nodes Ready
- Helm Release Installed
- AWS Load Balancer Controller Running
- Application Pods
- Envoy Gateway Pods
- Gateway Resource
- HTTPRoute
- Certificate Ready
- HTTPS Application Access

---

## Author

**Preetham**

**90 Days of DevOps Journey**

Focused on mastering DevOps, Cloud, Kubernetes, Terraform, CI/CD, and AWS through consistent hands-on learning and real-world projects.
