# Kubernetes Monitoring using Prometheus & Grafana on EKS

## Project Overview

This project provisions a complete Kubernetes monitoring setup using Terraform and Helm on Amazon EKS. It automates the deployment of Prometheus and Grafana using the kube-prometheus-stack Helm chart. Once deployed:

- Prometheus collects metrics from the Kubernetes nodes and pods.
- Grafana visualizes these metrics through pre-built dashboards.
- Grafana is also pre-configured with Prometheus as its data source.

---

## Tools and Services Used

| Tool/Service          | Purpose                               |
|-----------------------|---------------------------------------|
| AWS EKS               | Managed Kubernetes cluster            |
| Terraform             | Infrastructure provisioning           |
| kubectl               | Kubernetes CLI for cluster access     |
| Helm                  | Kubernetes package manager            |
| Prometheus            | Monitoring and alerting toolkit       |
| Grafana               | Dashboard for data visualization      |
| kube-prometheus-stack | Helm chart for integrated setup       |

---

## Prerequisites: 

- Terraform 
- AWS CLI configured with IAM user credentials
- kubectl installed and accessible

---

## How It Works

**1. Terraform:** provisions the complete EKS cluster with networking setup (VPC, subnets, security groups, etc.).
**2. Helm:** is used to install the `kube-prometheus-stack`, which includes:
   - Prometheus Server (metrics collection)
   - Grafana (visualization dashboard)
   - kube-state-metrics and node-exporter (cluster-level metrics exporters)
**3. Grafana Dashboards:** auto-imported from the Helm chart provide insights into cluster health and performance.
**4. Port-forwarding:** allows us to access Grafana and Prometheus locally.

---

## Architecture Diagram

<img width="1102" height="716" alt="Eks-monitoring-diagram" src="https://github.com/user-attachments/assets/ab0aea16-598b-4d98-963a-db35e8c94047" />

---

## Features

- One-click infrastructure provisioning with Terraform
- Monitoring stack installed using Helm charts
- Auto-configured Grafana dashboards
- Prometheus scraping system metrics via node-exporter
- Grafana pre-connected with Prometheus
- Modular, reusable infrastructure code

---

## Project Structure
```bash
├── terraform/
│ ├── main.tf
│ ├── variables.tf
│ ├── providers.tf
│ └── outputs.tf
├── .gitignore
└── README.md
```

---

## Steps to Run the Project:

**✅ Step 1: Clone the Repo**
```bash
git clone https://github.com/Sushmitha6300/eks-monitoring-stack-with-helm.git
cd eks-monitoring-stack-with-helm/terraform
```

**✅ Step 2: Provision Infrastructure using Terraform**
```bash
terraform init
```

Apply in Two Phases (Due to EKS Dependency)

**Phase 1:**

In providers.tf, comment out the kubernetes provider block and the data blocks:
```bash
# data "aws_eks_cluster" "cluster" {
#   name = module.eks.cluster_name
# }

# data "aws_eks_cluster_auth" "cluster" {
#   name = module.eks.cluster_name
# }

# provider "kubernetes" {
#   host = data.aws_eks_cluster.cluster.endpoint
#   cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority[0]data)
#   token = data.aws_eks_cluster_auth.cluster.token
# }
```

In main.tf, set this:
```bash
manage_aws_auth_configmap = false
```

Run:
```bash
terraform apply
```

**Phase 2:**

Uncomment the data blocks and Kubernetes provider block

In main.tf, change this:
```bash
manage_aws_auth_configmap = true
```
Run:
```bash
terraform apply
```

**✅ Step 3: Update your kubeconfig so kubectl can access the EKS cluster:**
```bash
aws eks --region us-east-1 --name eks-cluster update-kubeconfig
```
**✅ Step 4: Verify node group is registered:**
```bash
kubectl get nodes
```

**✅ Step 5: Install Helm**

Download and extract Helm manually:
```bash
curl -LO https://get.helm.sh/helm-v3.18.4-linux-amd64.tar.gz
tar -zxvf helm-v3.18.4-linux-amd64.tar.gz
```

Move Helm binary to a location in your PATH:
```bash
mv linux-amd64/helm ~/helm
chmod +x ~/helm
```

Add to PATH (temporary):
```bash
export PATH=$PATH:~
```

Check Helm is working:
```bash
helm version
```

**⚠️ Note:**  These steps are for Linux environments (including WSL on Windows). If you're using native Windows or macOS, download the appropriate version from the Helm releases page.

**✅ Step 6: Create a Namespace for Monitoring**

Create a namespace to logically group Prometheus, Grafana, and related resources.
```bash
kubectl create namespace monitoring
```

**✅ Step 7: Add Prometheus Community Helm Repo**

This repo provides the kube-prometheus-stack, which bundles all required tools.
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

**✅ Step 8: Install kube-prometheus-stack Helm Chart**
```bash
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

**✅ Step 9: Verify the Installation**

Check if the pods and services are up and running:
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

**✅ Step 10: Get Grafana Admin Password**

The default username is admin. Password is stored in a Kubernetes secret.
```bash
kubectl get secret prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

**✅ Step 11: Access Grafana UI**

Forward local port to the Grafana service port.
```bash
kubectl port-forward svc/prometheus-grafana -n monitoring 3000:80
```
Now let it run in the terminal, open your browser and visit:
```bash
http://localhost:3000
```

**Login with:**
- Username: admin
- Password: (copy from above command)

**✅ Step 12: Verify Prometheus as a Data Source in Grafana**

In Grafana, click the Connections → Data Sources

You should see Prometheus listed.

Scroll down and click on Test — it should say “✅ Data source is working”

**Explore Dashboards:**

Go to Dashboards and explore default dashboards:
- Kubernetes / Compute Resources / Cluster
- Kubernetes / Compute Resources / Node (Pods)
- Kubernetes / Networking / Cluster

Each dashboard gives detailed graphs and metrics.

**✅ Step 13: Access Prometheus UI**

If you want to access Prometheus directly:
```bash
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 -n monitoring
```

Visit: http://localhost:9090

You can try queries like:
```bash
node_memory_MemAvailable_bytes
```

to see memory available on nodes.

**✅ Alert Manager?**

We haven't configured custom alerts or receivers for Alertmanager in this project, although it was installed as part of the kube-prometheus-stack. You can extend the setup later with custom alert rules and routing logic.

### Final Notes:

- This is a production-grade monitoring stack for Kubernetes.
- Grafana dashboards are pre-built but can be customized.
- Alerting and Ingress can be added later for real-world use.

---

## Output

<img width="1920" height="970" alt="Grafana-homepage" src="https://github.com/user-attachments/assets/cc495b67-4cbc-498b-9372-3b9d85dccacd" />

<img width="1920" height="970" alt="Data-sources" src="https://github.com/user-attachments/assets/a4bd35b5-83fd-495f-a661-0d9c31b89d13" />

<img width="1920" height="959" alt="prometheus-metrics" src="https://github.com/user-attachments/assets/facbf06a-6bd1-49b1-b6fd-8bfdfb6e2f76" />

<img width="1920" height="954" alt="Prometheus-graph-metrics" src="https://github.com/user-attachments/assets/15b58e37-20c1-4023-b6a6-32ad9d1028b2" />

---

## About Me

Hey there! I’m Sushmitha, an aspiring DevOps Engineer passionate about automating infrastructure and streamlining deployments.

Currently, I’m building hands-on projects to master the DevOps lifecycle — from infrastructure as code to CI/CD and monitoring.

Always eager to learn, experiment, and take on new challenges in the cloud and DevOps world.

**Let’s connect!**

- LinkedIn: 
- GitHub: https://github.com/Sushmitha6300





