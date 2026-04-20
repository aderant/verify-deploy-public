# Verify Deploy - Multi-Cloud Infrastructure as Code

This repository contains the complete infrastructure setup for the Verify application, a document verification system that can be deployed to multiple cloud providers and generic Kubernetes clusters. The deployment supports Azure Kubernetes Service (AKS), Amazon EKS (placeholder), and generic Kubernetes clusters with flexible configuration options.

## 📋 Table of Contents

- [🚀 Quick Start Guide](#-quick-start-guide)
  - [Azure Deployment](#azure-deployment)
  - [Generic Kubernetes Deployment](#generic-kubernetes-deployment)
  - [AWS Deployment (Placeholder)](#aws-deployment-placeholder)
- [🏗️ Architecture Overview](#️-architecture-overview)
- [🚀 Supported Deployment Scenarios](#-supported-deployment-scenarios)
- [📁 Repository Structure](#-repository-structure)
- [📋 Configuration Guide](#-configuration-guide)
- [🔧 Advanced Configuration](#-advanced-configuration)
- [🔧 Manual Deployment](#-manual-deployment)
- [🔐 Security Features](#-security-features)
- [🐳 Container Registry](#-container-registry)
- [📦 Helm Charts](#-helm-charts)
- [🔧 Cluster Access](#-cluster-access)
- [🛠️ Troubleshooting](#️-troubleshooting)
- [🎯 Common Deployment Scenarios](#-common-deployment-scenarios)
- [📚 Best Practices](#-best-practices)
- [🔄 Maintenance and Updates](#-maintenance-and-updates)
- [📊 Monitoring and Logging](#-monitoring-and-logging)
- [🔄 Migration Guide](#-migration-guide)
- [🏗️ Bastion Host Implementation](#️-bastion-host-implementation)
- [🔐 External Secrets Integration](#-external-secrets-integration)
- [📋 Scripts Overview](#-scripts-overview)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)
- [🆘 Support](#-support)

## 🚀 Quick Start Guide

Choose your deployment method based on your infrastructure needs:

### Azure Deployment

Complete deployment to Azure Kubernetes Service (AKS) with managed infrastructure.

#### Prerequisites

**Required Tools:**
- **Azure CLI** (v2.50.0+): [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- **Terraform** (v1.5.0+): [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- **Helm** (v3.12.0+): [Install Helm](https://helm.sh/docs/intro/install/)
- **kubectl** (v1.28.0+): [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
- **Python 3.8+** with PyYAML: `pip install PyYAML`

**Azure Setup:**
```bash
# Login to Azure
az login

# Set your subscription
az account set --subscription "your-subscription-id"

# Verify access
az account show
```

#### 1. Clone and Configure

```bash
# Clone the repository
git clone <repository-url>
cd verify-deploy

# Copy the example configuration
cp providers/azure/terraform/terraform.tfvars.example providers/azure/terraform/terraform.tfvars

# Edit the configuration with your values
nano providers/azure/terraform/terraform.tfvars
```

**Required Configuration Values:**
- `subscription_id`: Your Azure subscription ID
- `project_name`: Your project name (e.g., "verify")
- `environment`: Environment name (e.g., "dev", "prod")
- `location`: Azure region (e.g., "eastus")

#### 2. Deploy Infrastructure

```bash
# Deploy Azure infrastructure (AKS, database, storage, etc.)
./providers/azure/scripts/deploy-infra.sh
```

This script will:
- Create Azure resource groups
- Deploy AKS cluster with GPU and CPU node pools
- Set up PostgreSQL database
- Create storage accounts and blob containers
- Configure Azure Key Vault for secrets management
- Set up networking and security

#### 3. Deploy Application

```bash
# Deploy the Verify application
./providers/azure/scripts/deploy-helm.sh
```

This script will:
- Generate Helm values from Terraform outputs
- Deploy External Secrets Operator
- Deploy the Verify application components
- Configure ingress and TLS certificates

For an existing AKS cluster where you only want to test a new packaged chart version, you can skip the Terraform-dependent bootstrap steps:

```bash
./providers/azure/scripts/deploy-helm.sh \
  --upgrade-only \
  --cluster-name <aks-name> \
  --resource-group <aks-resource-group> \
  --chart-package helm-charts/verify/verify-<version>.tgz \
  --values my-values.yaml
```

#### 4. Verify Deployment

```bash
# Run comprehensive validation
./scripts/validate.sh

# Check application status
kubectl get pods -n verify
kubectl get services -n verify
```

#### 5. Access the Application

```bash
# Get the application URL
kubectl get ingress -n verify

# The application will be available at the hostname shown in the ingress
```

### Generic Kubernetes Deployment

Deploy to an existing Kubernetes cluster (any provider).

#### Prerequisites

**Required Tools:**
- **Helm** (v3.12.0+): [Install Helm](https://helm.sh/docs/intro/install/)
- **kubectl** (v1.28.0+): [Install kubectl](https://kubernetes.io/docs/tasks/tools/)
- **Existing Kubernetes cluster** with appropriate resources

#### 1. Configure Values

```bash
# Copy and customize the generic configuration
cp providers/generic/values-generic-example.yaml my-values.yaml
nano my-values.yaml
```

#### 2. Validate Configuration

```bash
# Validate your configuration
./scripts/validate-config.sh --values my-values.yaml
```

#### 3. Deploy Application

```bash
# Deploy the application
./providers/generic/scripts/deploy-helm.sh --values my-values.yaml
```

### AWS Deployment (Placeholder)

Deploy to Amazon EKS (future implementation).

#### Prerequisites

**Required Tools:**
- **AWS CLI** (v2.0+): [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- **Terraform** (v1.5.0+): [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- **Helm** (v3.12.0+): [Install Helm](https://helm.sh/docs/intro/install/)
- **kubectl** (v1.28.0+): [Install kubectl](https://kubernetes.io/docs/tasks/tools/)

**AWS Setup:**
```bash
# Configure AWS CLI
aws configure

# Verify access
aws sts get-caller-identity
```

#### 1. Configure Values

```bash
# Copy the AWS example configuration
cp providers/aws/values-aws-example.yaml my-values.yaml
nano my-values.yaml
```

#### 2. Deploy Infrastructure

```bash
# Deploy AWS infrastructure (EKS, RDS, S3, etc.)
./providers/aws/scripts/deploy-infra.sh
```

#### 3. Deploy Application

```bash
# Deploy the Verify application
./providers/aws/scripts/deploy-helm.sh
```

**Note**: AWS deployment is currently a placeholder. The infrastructure and application deployment scripts are not yet implemented.

## 🏗️ Architecture Overview

The infrastructure consists of three main layers:

- **Compute Layer**: Kubernetes cluster with GPU and CPU node pools
- **Storage Layer**: PostgreSQL database and object storage with provider-specific implementations
- **Application Layer**: Kubernetes deployments for UI, verification, inference, DPS, and DSS services

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Azure Cloud                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   AKS Cluster   │  │   PostgreSQL    │  │ Storage Account │  │
│  │                 │  │   Database      │  │                 │  │
│  │ ┌─────────────┐ │  │                 │  │ ┌─────────────┐ │  │
│  │ │ UI Service  │ │  │ ┌─────────────┐ │  │ │ Blob Store  │ │  │
│  │ └─────────────┘ │  │ │ Extraction  │ │  │ │ - Documents │ │  │
│  │                 │  │ │ Database    │ │  │ │ - Weights   │ │  │
│  │ ┌─────────────┐ │  │ └─────────────┘ │  │ └─────────────┘ │  │
│  │ │Verification │ │  └─────────────────┘  └─────────────────┘  │
│  │ │Service      │ │                                            │
│  │ └─────────────┘ │  ┌─────────────────┐                      │
│  │                 │  │ Private DNS     │                      │
│  │ ┌─────────────┐ │  │ Zones           │                      │
│  │ │ Inference   │ │  └─────────────────┘                      │
│  │ │ Service     │ │                                            │
│  │ └─────────────┘ │                                            │
│  │                 │                                            │
│  │ ┌─────────────┐ │                                            │
│  │ │ DPS Service │ │                                            │
│  │ └─────────────┘ │                                            │
│  │                 │                                            │
│  │ ┌─────────────┐ │                                            │
│  │ │ DSS Service │ │                                            │
│  │ └─────────────┘ │                                            │
│  └─────────────────┘                                            │
└─────────────────────────────────────────────────────────────────┘
```

### Components

#### 1. Compute Layer (AKS)
- **Azure Kubernetes Service Cluster**: Container orchestration platform
- **Configuration**: Private cluster with Azure CNI networking
- **Node Pools**:
  - System pool: Default system workloads
  - CPU pool: CPU-intensive workloads (verification service)
  - GPU pool: GPU-accelerated workloads (inference service)

#### 2. Storage Layer
- **PostgreSQL Flexible Server**: Primary database for application data
- **Azure Storage Account**: Blob storage for documents and model weights
- **Private Endpoints**: Secure connectivity to storage services

#### 3. Application Layer
- **UI Service**: Web interface for document verification
- **Verification Service**: Document processing and verification logic
- **Inference Service**: AI/ML model inference for document analysis
- **DPS Service**: Document processing service
- **Data Sync Service (DSS)**: Synchronizes data between external systems (E3, Aderant)

## 🚀 Supported Deployment Scenarios

### 1. Azure Kubernetes Service (AKS) - Production Ready
- **Infrastructure**: Terraform-managed AKS cluster with private networking
- **Storage**: Azure Database for PostgreSQL and Azure Storage Account
- **Secrets**: External Secrets Operator with Azure Key Vault integration
- **Networking**: Private endpoints and internal load balancers
- **GPU Support**: NVIDIA device plugin for GPU workloads

### 2. Generic Kubernetes Cluster - Production Ready
- **Infrastructure**: Existing Kubernetes cluster (any provider)
- **Storage**: In-cluster PostgreSQL or external database
- **Secrets**: Standard Kubernetes secrets or External Secrets Operator
- **Networking**: NGINX Ingress Controller with LoadBalancer service
- **GPU Support**: NVIDIA device plugin (optional)

### 3. Amazon EKS - Placeholder (Future Implementation)
- **Infrastructure**: Terraform-managed EKS cluster
- **Storage**: Amazon RDS and S3
- **Secrets**: External Secrets Operator with AWS Secrets Manager
- **Networking**: AWS Load Balancer Controller
- **GPU Support**: NVIDIA device plugin for GPU workloads

## 📁 Repository Structure

```
├── providers/                           # Cloud provider-specific configurations
│   ├── azure/                          # Azure-specific deployment
│   │   ├── terraform/                  # Azure infrastructure as code
│   │   │   ├── main.tf                 # Main Terraform configuration
│   │   │   ├── variables.tf            # Variable definitions
│   │   │   ├── outputs.tf              # Output definitions
│   │   │   ├── terraform.tfvars.example # Example configuration
│   │   │   └── modules/                # Terraform modules
│   │   ├── scripts/                    # Azure deployment scripts
│   │   │   ├── deploy-infra.sh         # Deploy infrastructure
│   │   │   ├── deploy-helm.sh          # Deploy application
│   │   │   ├── generate-*.sh           # Generate configuration files
│   │   │   └── kubectl-invoke.sh       # Private cluster access helper
│   │   └── values-azure-example.yaml   # Example Azure values
│   ├── aws/                            # AWS-specific deployment (placeholder)
│   │   ├── terraform/                  # AWS infrastructure as code
│   │   ├── scripts/                    # AWS deployment scripts
│   │   └── values-aws-example.yaml     # Example AWS values
│   └── generic/                        # Generic Kubernetes deployment
│       ├── scripts/                    # Generic deployment scripts
│       │   └── deploy-helm.sh          # Deploy to generic cluster
│       ├── values-generic-example.yaml # Example generic values
│       └── customValuesGeneric.yaml    # Generic custom values
├── helm-charts/                        # Helm charts for application deployment
│   ├── verify/                         # Main application chart
│   │   ├── Chart.yaml                  # Chart metadata
│   │   ├── values.yaml                 # Default values
│   │   ├── templates/                  # Kubernetes templates
│   │   └── charts/                     # Chart dependencies
│   ├── storage/                        # Storage layer subchart
│   │   ├── Chart.yaml                  # Chart metadata
│   │   ├── values.yaml                 # Default values
│   │   └── templates/                  # Storage templates
│   └── application/                    # Application layer subchart
│       ├── Chart.yaml                  # Chart metadata
│       ├── values.yaml                 # Default values
│       └── templates/                  # Application templates
├── scripts/                            # Shared deployment and utility scripts
│   ├── validate-config.sh              # Configuration validation
│   ├── validate.sh                     # Deployment validation
│   ├── validate-coverage.sh            # Coverage validation
│   ├── migrate.sh                      # Migration utilities
│   ├── migrate-to-multi-cloud.sh       # Multi-cloud migration
│   └── restrict-keyvault-access.sh     # Security utilities
├── docs/                               # Documentation
│   └── troubleshooting.md              # Detailed troubleshooting guide
├── .gitignore                          # Git ignore rules
└── README.md                           # This file
```

## 📋 Configuration Guide

### Understanding Configuration Files

The deployment uses a hierarchical configuration system:

1. **Terraform Configuration** (`terraform.tfvars`): Infrastructure settings
2. **Helm Values** (`values.yaml`): Application configuration
3. **External Secrets** (`external-secrets-values.yaml`): Secret management

### Azure Configuration

#### 1. Terraform Configuration

Copy and customize the example:
```bash
cp providers/azure/terraform/terraform.tfvars.example providers/azure/terraform/terraform.tfvars
```

**Required Values:**
```hcl
# Basic Configuration
project_name     = "verify"                    # Your project name
environment      = "dev"                       # Environment (dev, staging, prod)
location         = "eastus"                    # Azure region
subscription_id  = "your-subscription-id"      # Your Azure subscription ID

# Cluster Configuration
cluster_name     = "verify-aks"                # AKS cluster name
cluster_type     = "GPU"                       # GPU or CPU cluster
kubernetes_version = "1.28.0"                 # Kubernetes version

# Node Pool Configuration
default_node_count = 2                         # System node count
default_vm_size    = "Standard_D2s_v3"        # System node size
cpu_vm_size        = "Standard_D32as_v5"      # CPU node size
gpu_vm_size        = "Standard_NC24ads_A100_v4"       # GPU node size (if cluster_type = "GPU")
```

#### 2. Application Configuration

The application configuration is automatically generated from Terraform outputs, but you can override specific values by creating a custom values file:

```bash
# Create custom values override
cat > my-custom-values.yaml << EOF
application:
  ui:
    image:
      tag: "specific-version"
  inference:
    resources:
      limits:
        nvidia.com/gpu: "2"
EOF

# Deploy with custom values (after running generate-helm-values-updated.sh)
helm upgrade verify ./helm-charts/verify \
  --values providers/azure/helm-values-generated.yaml \
  --values providers/azure/external-secrets-values.yaml \
  --values my-custom-values.yaml
```

### Generic Kubernetes Configuration

#### 1. Create Values File

```bash
cp providers/generic/values-generic-example.yaml my-values.yaml
```

#### 2. Configure Required Values

```yaml
# Global Configuration
global:
  namespace: verify
  imageRegistry: your-registry.com/verify
  imagePullSecrets:
    - name: your-registry-auth

# Provider Configuration
provider:
  type: "generic"
  generic:
    useExternalSecrets: false
    useManagedStorage: false
    useInClusterDatabase: true

# Database Configuration
database:
  mode: "in-cluster"
  inCluster:
    enabled: true
    auth:
      postgresPassword: "secure-password"
      database: "verify"
      username: "verify"
      password: "secure-password"

# Application Configuration
application:
  ingress:
    enabled: true
    hosts:
      - host: your-domain.com
        paths:
          - path: /
            pathType: Prefix
            service: ui
```

#### 3. Validate Configuration

```bash
# Validate your configuration
./scripts/validate-config.sh --values my-values.yaml

# Check for common issues
./scripts/validate-config.sh --check-required --values my-values.yaml
```

### Example Configuration Files

The repository includes comprehensive example configuration files to help you get started:

#### Azure Examples
- **`providers/azure/terraform/terraform.tfvars.example`**: Basic Azure configuration
- **`providers/azure/values-azure-example.yaml`**: Azure-specific Helm values

#### Generic Kubernetes Examples
- **`providers/generic/values-generic-example.yaml`**: Generic Kubernetes configuration
- **`providers/generic/customValuesGeneric.yaml`**: Custom values example

#### AWS Examples (Placeholder)
- **`providers/aws/values-aws-example.yaml`**: AWS-specific configuration (future implementation)

#### Usage
```bash
# Copy and customize the appropriate example
cp providers/azure/terraform/terraform.tfvars.example providers/azure/terraform/terraform.tfvars
cp providers/azure/values-azure-example.yaml my-values.yaml

# Edit with your specific values
nano providers/azure/terraform/terraform.tfvars
nano my-values.yaml
```

## 🔧 Advanced Configuration

### Provider Selection

The deployment behavior is controlled by the `provider.type` setting in your values file:

```yaml
provider:
  type: "azure"    # Options: azure, aws, generic
```

### Database Configuration

#### In-Cluster Database (Generic/AWS)
```yaml
database:
  mode: "in-cluster"
  inCluster:
    enabled: true
    auth:
      postgresPassword: "secure-password"
      database: "verify"
      username: "verify"
      password: "secure-password"
    persistence:
      size: 20Gi
      storageClass: "default"
```

#### External Database (Azure/AWS)
```yaml
database:
  mode: "external"
  external:
    enabled: true
    host: "your-database-host"
    port: 5432
    database: "verify"
    username: "verify"
    password: ""  # Will be populated from secrets
    sslMode: "require"
```

### TLS Configuration with cert-manager

#### Automatic TLS Certificate Management
When cert-manager is enabled, the ingress will automatically request and manage TLS certificates from Let's Encrypt:

```yaml
# Enable cert-manager
certManager:
  enabled: true
  namespace: cert-manager

# Configure ClusterIssuer for Let's Encrypt
clusterIssuer:
  name: letsencrypt-prod
  acme:
    email: "your-email@example.com"  # Required: your email for Let's Encrypt registration
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
      - http01:
          ingress:
            class: azure-application-gateway  # For Azure Application Gateway

# Configure ingress with TLS
application:
  ingress:
    enabled: true
    hosts:
      - host: "your-domain.com"  # Your actual domain name
        paths:
          - path: /
            pathType: Prefix
            service: ui
    tls:
      - secretName: tls-secret
        hosts:
          - your-domain.com
```

#### Manual TLS Configuration (without cert-manager)
```yaml
# Disable cert-manager
certManager:
  enabled: false

# Configure ingress with manual TLS secret
application:
  ingress:
    enabled: true
    hosts:
      - host: "your-domain.com"
        paths:
          - path: /
            pathType: Prefix
            service: ui
    tls:
      - secretName: tls-secret  # You must create this secret manually
        hosts:
          - your-domain.com
```

### Secrets Management

#### External Secrets (Azure)
```yaml
secrets:
  mode: "external-secrets"
externalSecrets:
  enabled: true
  # Azure Key Vault configuration
```

#### Kubernetes Secrets (Generic/AWS)
```yaml
secrets:
  mode: "kubernetes-secrets"
  kubernetesSecrets:
    enabled: true
    database:
      name: "db"
      data:
        host: "your-database-host"
        database: "verify"
        username: "verify"
        password: "your-password"
```

### Infrastructure Components

#### NVIDIA Device Plugin
```yaml
infrastructure:
  nvidiaDevicePlugin:
    enabled: true  # Enable for GPU support
    image:
      repository: nvcr.io/nvidia/k8s-device-plugin
      tag: "v0.17.2"
```

#### NGINX Ingress Controller
```yaml
infrastructure:
  nginxIngress:
    enabled: true
    controller:
      service:
        type: LoadBalancer
        annotations:
          # Provider-specific annotations
          service.beta.kubernetes.io/azure-load-balancer-internal: "true"  # Azure
          service.beta.kubernetes.io/aws-load-balancer-type: "nlb"         # AWS
```

#### cert-manager (Optional)
```yaml
# cert-manager configuration for automatic TLS certificate management
certManager:
  enabled: true  # Set to true to enable cert-manager installation
  namespace: cert-manager  # Namespace for cert-manager installation
  values:
    crds:
      enabled: true
    # Additional cert-manager configuration can be added here
    # Example: prometheus.enabled, webhook.timeoutSeconds, etc.

# ClusterIssuer configuration (automatically enabled when certManager.enabled is true)
clusterIssuer:
  name: letsencrypt-prod
  acme:
    email: "your-email@example.com"  # Required: your email for Let's Encrypt registration
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
      - http01:
          ingress:
            class: azure-application-gateway  # For Azure Application Gateway
```

**Note**: The ClusterIssuer is created using a Helm post-install hook to ensure cert-manager CRDs are installed first. This prevents installation failures due to missing CRDs.

## 🔧 Manual Deployment

### Azure Manual Deployment

1. **Deploy Infrastructure**:
   ```bash
   cd providers/azure/terraform
   terraform init
   terraform plan
   terraform apply
   ```

2. **Deploy Applications**:
   ```bash
   # Generate Helm values from Terraform outputs
   ./providers/azure/scripts/generate-helm-values-updated.sh
   
   # Deploy using Helm (after generating values)
   helm install verify ./helm-charts/verify \
     --values providers/azure/helm-values-generated.yaml \
     --values providers/azure/external-secrets-values.yaml
   ```

### Generic Kubernetes Manual Deployment

1. **Prepare values file**:
   ```bash
   cp providers/generic/values-generic-example.yaml my-values.yaml
   # Edit my-values.yaml with your configuration
   ```

2. **Deploy using Helm**:
   ```bash
   helm install verify ./helm-charts/verify \
     --namespace verify \
     --create-namespace \
     --values my-values.yaml
   ```

## 🔐 Security Features

### Azure Deployment
- Private AKS cluster with no public API server
- Private endpoints for storage and database
- Network policies for pod-to-pod communication
- RBAC enabled with least privilege access
- **ExternalSecrets integration with Azure Key Vault**
- **Workload Identity for secure secret access**
- **No secrets stored in Kubernetes directly**
- **ACR authentication with service principal**

### Generic Kubernetes Deployment
- Network policies for pod-to-pod communication
- RBAC enabled with least privilege access
- **Standard Kubernetes secrets or External Secrets Operator**
- **Image pull secrets for private registries**
- **TLS termination at ingress controller**

## 🐳 Container Registry

The application supports multiple container registries with:
- **Primary Registry**: `aderanteng.azurecr.io` (Aderant ACR)
- **Secondary Registry**: `zerosystems.azurecr.io` (Hercules ACR)
- **Custom Registries**: Configurable per deployment
- **Default image tags**: `latest` (for easy updates)
- **Registry override capability**: Use different registries per component
- **Version override capability**: Specify exact versions when needed
- **Multi-registry authentication**: Support for multiple registries simultaneously

### Registry Override Examples

#### Component-Specific Registry Overrides
```yaml
application:
  dps:
    image:
      registry: zerosystems.azurecr.io/zerosystems/verify
      repository: document-preprocessing-service
      tag: "0.0.535"
  inference:
    image:
      registry: aderanteng.azurecr.io/zerosystems/verify
      repository: inference
      tag: "latest"
```

## 📦 Helm Charts

The application is deployed using Helm charts with the following structure:

### Main Chart: `verify`
- **Purpose**: Main application chart that orchestrates all components
- **Dependencies**: `storage` and `application` subcharts
- **Location**: `helm-charts/verify/`

### Subcharts:
- **`storage`**: Redis, RabbitMQ, and blob storage components
- **`application`**: DPS, Inference, Verification, and UI services

### Key Features:
- **Modular design**: Separate charts for different concerns
- **Value inheritance**: Global values passed to subcharts
- **Override capability**: Easy to customize specific components
- **Provider-specific templates**: Conditional deployment based on provider type
- **Infrastructure components**: NVIDIA device plugin and NGINX ingress controller
- **Flexible secrets management**: External Secrets or Kubernetes secrets
- **Database options**: In-cluster or external PostgreSQL

## 🔧 Cluster Access

### Azure Private Cluster Access
Since the AKS cluster is private, use the provided helper script for kubectl operations:

```bash
# View pods
./providers/azure/scripts/kubectl-invoke.sh "get pods" verify

# View services
./providers/azure/scripts/kubectl-invoke.sh "get services" verify

# View logs
./providers/azure/scripts/kubectl-invoke.sh "logs <pod-name>" verify
```

### Generic Kubernetes Cluster Access
Use standard kubectl commands with your current kubecontext:

```bash
# View pods
kubectl get pods -n verify

# View services
kubectl get services -n verify

# View logs
kubectl logs -l app.kubernetes.io/instance=verify -n verify

# Port forward
kubectl port-forward -n verify service/verify-ui 8080:80
```

## 🛠️ Troubleshooting

### Quick Diagnostics

Run these commands to quickly diagnose common issues:

```bash
# 1. Check overall system health
./scripts/validate.sh

# 2. Validate your configuration
./scripts/validate-config.sh --values your-values.yaml

# 3. Check cluster connectivity
kubectl cluster-info
kubectl get nodes

# 4. Check application status
kubectl get pods -n verify
kubectl get services -n verify
kubectl get ingress -n verify

# 5. Check for errors
kubectl get events -n verify --sort-by='.lastTimestamp'
```

### Common Issues and Solutions

#### 1. Deployment Failures

**Issue**: Infrastructure deployment fails
```bash
# Check Terraform state
cd providers/azure/terraform
terraform plan
terraform show

# Check Azure permissions
az account show
az role assignment list --assignee $(az account show --query user.name -o tsv)
```

**Issue**: Application deployment fails
```bash
# Check Helm chart syntax
helm template verify ./helm-charts/verify --values your-values.yaml

# Check for missing secrets
kubectl get secrets -n verify
kubectl describe secret <secret-name> -n verify
```

#### 2. Pod Issues

**Issue**: Pods stuck in Pending
```bash
# Check node resources
kubectl describe nodes
kubectl top nodes

# Check resource quotas
kubectl describe quota -n verify

# Check persistent volume claims
kubectl get pvc -n verify
kubectl describe pvc <pvc-name> -n verify
```

**Issue**: Pods in CrashLoopBackOff
```bash
# Check pod logs
kubectl logs <pod-name> -n verify --previous
kubectl describe pod <pod-name> -n verify

# Check resource limits
kubectl top pods -n verify
kubectl describe pod <pod-name> -n verify | grep -A 10 "Limits:"
```

#### 3. Network Issues

**Issue**: Services not accessible
```bash
# Check service endpoints
kubectl get endpoints -n verify
kubectl describe service <service-name> -n verify

# Test internal connectivity
kubectl exec -it <pod-name> -n verify -- curl http://<service-name>:<port>

# Check ingress configuration
kubectl describe ingress -n verify
```

**Issue**: DNS resolution problems
```bash
# Check DNS configuration
kubectl get configmap -n kube-system coredns
kubectl exec -it <pod-name> -n verify -- nslookup kubernetes.default
```

#### 4. Storage Issues

**Issue**: PVC stuck in Pending
```bash
# Check storage class
kubectl get storageclass
kubectl describe storageclass <storage-class-name>

# Check Azure storage account (for Azure deployments)
az storage account show --name <storage-account> --resource-group <rg>
```

**Issue**: Mount errors
```bash
# Check storage account connectivity
kubectl exec -it <pod-name> -n verify -- nslookup <storage-account>.blob.core.windows.net

# Check private endpoint (for Azure)
az network private-endpoint show --name <endpoint-name> --resource-group <rg>
```

#### 5. Database Issues

**Issue**: Database connection failed
```bash
# Check database secret
kubectl get secret db -n verify -o yaml
kubectl describe secret db -n verify

# Test database connectivity
kubectl exec -it <pod-name> -n verify -- nslookup <db-host>
kubectl exec -it <pod-name> -n verify -- psql -h <db-host> -U <user> -d <database>
```

#### 6. GPU Issues

**Issue**: GPU not available
```bash
# Check GPU nodes
kubectl get nodes -l model_device=gpu
kubectl describe nodes -l model_device=gpu

# Check NVIDIA device plugin
kubectl get pods -n kube-system -l name=nvidia-device-plugin-ds
kubectl logs -n kube-system -l name=nvidia-device-plugin-ds
```

#### 7. cert-manager Issues

**Issue**: Certificate not issued
```bash
# Check cert-manager status
kubectl get pods -n cert-manager
kubectl get clusterissuers
kubectl describe clusterissuer letsencrypt-prod

# Check certificate status
kubectl get certificates --all-namespaces
kubectl describe certificate <cert-name> -n verify
```

**Issue**: HTTP01 challenge failing
```bash
# Check ingress configuration
kubectl describe ingress -n verify
kubectl get ingress -n verify -o yaml

# Check cert-manager logs
kubectl logs -n cert-manager -l app=cert-manager
```

### Advanced Diagnostics

#### Complete System Health Check
```bash
#!/bin/bash
echo "=== Cluster Status ==="
kubectl cluster-info
kubectl get nodes

echo "=== Pod Status ==="
kubectl get pods -n verify

echo "=== Service Status ==="
kubectl get services -n verify

echo "=== Storage Status ==="
kubectl get pvc -n verify
kubectl get pv

echo "=== Resource Usage ==="
kubectl top nodes
kubectl top pods -n verify

echo "=== Events ==="
kubectl get events -n verify --sort-by='.lastTimestamp'
```

#### Resource Investigation
```bash
# Check resource quotas and limits
kubectl describe quota -n verify
kubectl describe nodes
kubectl top pods -n verify --containers

# Check storage usage
kubectl exec -it <pod-name> -n verify -- df -h
```

#### Network Investigation
```bash
# Check network connectivity
kubectl exec -it <pod-name> -n verify -- ping <target-ip>
kubectl exec -it <pod-name> -n verify -- nslookup <hostname>
kubectl exec -it <pod-name> -n verify -- telnet <hostname> <port>

# Check network policies
kubectl get networkpolicy -n verify
kubectl describe networkpolicy <policy-name> -n verify
```

### Getting Help

If you're still experiencing issues:

1. **Check the detailed troubleshooting guide**: `docs/troubleshooting.md`
2. **Run validation scripts**: `./scripts/validate.sh` and `./scripts/validate-config.sh`
3. **Check application logs**: `kubectl logs -l app.kubernetes.io/instance=verify -n verify`
4. **Review system events**: `kubectl get events -n verify --sort-by='.lastTimestamp'`
5. **Create an issue**: Include the output of the diagnostic commands above

## 🎯 Common Deployment Scenarios

### Development Environment
For development and testing:
```bash
# Copy and customize the basic configuration
cp providers/azure/terraform/terraform.tfvars.example providers/azure/terraform/terraform.tfvars

# Edit with your values (set cluster_type = "CPU" for cost savings)
nano providers/azure/terraform/terraform.tfvars

# Deploy
./providers/azure/scripts/deploy-infra.sh
./providers/azure/scripts/deploy-helm.sh
```

### Production Environment
For production deployments:
```bash
# Copy and customize the basic configuration
cp providers/azure/terraform/terraform.tfvars.example providers/azure/terraform/terraform.tfvars

# Edit with production values (set cluster_type = "GPU" for full functionality)
nano providers/azure/terraform/terraform.tfvars

# Deploy with specific versions
./providers/azure/scripts/deploy-infra.sh
./providers/azure/scripts/deploy-helm.sh
```

### Multi-Registry Deployment
For using multiple container registries:
```bash
# Create custom values with multi-registry configuration
cat > my-multi-registry-values.yaml << EOF
application:
  dps:
    image:
      registry: zerosystems.azurecr.io/zerosystems/verify
      repository: document-preprocessing-service
      tag: "0.0.535"
  inference:
    image:
      registry: aderanteng.azurecr.io/zerosystems/verify
      repository: inference
      tag: "latest"
EOF

# Deploy with custom values
helm upgrade verify ./helm-charts/verify \
  --values providers/azure/helm-values-generated.yaml \
  --values providers/azure/external-secrets-values.yaml \
  --values my-multi-registry-values.yaml
```

### Generic Kubernetes Deployment
For existing Kubernetes clusters:
```bash
# Use generic configuration
cp providers/generic/values-generic-example.yaml my-values.yaml

# Edit with your cluster details
nano my-values.yaml

# Deploy
./providers/generic/scripts/deploy-helm.sh --values my-values.yaml
```

## 📚 Best Practices

### Security
- **Never commit sensitive files**: Use `.gitignore` to exclude `terraform.tfvars`, `values.yaml`, and other sensitive files
- **Use External Secrets**: Prefer Azure Key Vault integration over hardcoded secrets
- **Private clusters**: Use private AKS clusters for production
- **Network policies**: Implement network policies for pod-to-pod communication

### Resource Management
- **Resource limits**: Always set appropriate resource limits and requests
- **Node selectors**: Use node selectors to place workloads on appropriate nodes
- **Storage classes**: Use appropriate storage classes for your workload requirements
- **Monitoring**: Set up monitoring and alerting for production deployments

### Configuration Management
- **Version control**: Keep configuration files in version control (excluding sensitive data)
- **Environment separation**: Use different configurations for different environments
- **Validation**: Always validate configuration before deployment
- **Documentation**: Document any custom configurations or overrides

### Deployment Strategy
- **Staged deployment**: Test in development before production
- **Rolling updates**: Use rolling updates for zero-downtime deployments
- **Backup strategy**: Implement backup and disaster recovery procedures
- **Health checks**: Configure proper health checks and probes

## 🔄 Maintenance and Updates

### Updating the Application
```bash
# Update to latest versions (after generating values)
helm upgrade verify ./helm-charts/verify \
  --values providers/azure/helm-values-generated.yaml \
  --values providers/azure/external-secrets-values.yaml

# Update to specific versions
helm upgrade verify ./helm-charts/verify \
  --values providers/azure/helm-values-generated.yaml \
  --values providers/azure/external-secrets-values.yaml \
  --values my-custom-versions.yaml
```

### One-time database cleanup before upgrade
For test environments, you can run a one-off database cleanup job before the Helm upgrade by enabling the chart hook:

```bash
helm upgrade verify ./helm-charts/verify \
  --values providers/azure/helm-values-generated.yaml \
  --values providers/azure/external-secrets-values.yaml \
  --set databaseCleanup.enabled=true \
  --set-string databaseCleanup.confirmDestructive=WIPE_DATABASE
```

The hook runs before the upgrade and deletes itself on success. For external database deployments it reads connection settings from the `db` secret; when `database.mode=in-cluster`, it uses `postgres-secret`. The destructive job is rendered only when `databaseCleanup.confirmDestructive=WIPE_DATABASE` is set alongside `databaseCleanup.enabled=true`. Turn `databaseCleanup.enabled` back off after that run so later upgrades do not wipe the database again.

### Updating Infrastructure
```bash
# Update Terraform configuration
cd providers/azure/terraform
terraform plan
terraform apply

# Regenerate Helm values
./providers/azure/scripts/generate-helm-values-updated.sh

# Update application
./providers/azure/scripts/deploy-helm.sh
```

### Monitoring and Maintenance
```bash
# Check system health
./scripts/validate.sh

# Monitor resource usage
kubectl top nodes
kubectl top pods -n verify

# Check for updates
helm list -n verify
kubectl get pods -n verify
```

## 📊 Monitoring and Logging

### Azure Deployment
- **Azure Monitor**: Integrated with AKS for metrics and logs
- **Application Insights**: For application performance monitoring
- **Log Analytics**: Centralized logging

### Generic Kubernetes Deployment
- **Standard Kubernetes**: Use your cluster's monitoring solution
- **Prometheus/Grafana**: Recommended for metrics and dashboards
- **ELK Stack**: Recommended for centralized logging

## 🔄 Migration Guide

### From Azure-Only to Multi-Cloud

1. **Backup existing configuration**:
   ```bash
   # Backup your current Terraform state and values
   cp providers/azure/terraform/terraform.tfvars backup-terraform.tfvars
   ```

2. **Update to new structure**:
   ```bash
   # Your existing Azure deployment will continue to work
   # No changes required for existing Azure deployments
   ```

3. **Deploy to new provider**:
   ```bash
   # For generic Kubernetes
   ./providers/generic/scripts/deploy-helm.sh --provider generic --values my-values.yaml
   ```

### Updating Existing Deployments

1. **Backup Current Configuration**:
   ```bash
   kubectl get all -n verify -o yaml > backup-verify-resources.yaml
   ```

2. **Update Values File**:
   ```bash
   # Add new configuration options to your values file
   ```

3. **Upgrade Deployment**:
   ```bash
   helm upgrade verify ./helm-charts/verify --values your-values.yaml
   ```

## 🏗️ Bastion Host Implementation

The deployment includes an optional Ubuntu VM bastion host for secure access to private AKS clusters.

### Features
- **Ubuntu 22.04 LTS** VM with pre-installed tools (Azure CLI, kubectl, Helm)
- **SSH Key Authentication** with generated RSA 4096-bit key pair
- **Public IP Address** for external access
- **Dedicated Subnet** with Network Security Group
- **SSH Keys Stored in Key Vault** for secure retrieval

### Usage
```bash
# Download SSH private key
KEY_VAULT_NAME=$(terraform output -raw key_vault_name)
az keyvault secret show --vault-name $KEY_VAULT_NAME --name bastion-private-key --query value -o tsv > bastion_key.pem
chmod 600 bastion_key.pem

# Start the bastion VM
RESOURCE_GROUP=$(terraform output -raw networking_resource_group)
VM_NAME=$(terraform output -raw bastion_vm_name)
az vm start --resource-group $RESOURCE_GROUP --name $VM_NAME

# Connect to bastion host
PUBLIC_IP=$(terraform output -raw bastion_public_ip)
USERNAME=$(terraform output -raw bastion_connection_info | jq -r '.username')
ssh -i bastion_key.pem $USERNAME@$PUBLIC_IP
```

## 🔐 External Secrets Integration

The deployment uses External Secrets Operator for secure secret management.

### Architecture
```
Azure Key Vault
    ↓ (Workload Identity)
External Secrets Operator
    ↓ (SecretStore + ExternalSecret)
Kubernetes Secrets
    ↓ (secretRef)
Application Pods
```

### Benefits
- **Centralized Secret Management**: All secrets managed in Azure Key Vault
- **Automatic Synchronization**: Secrets automatically synced to Kubernetes
- **Security**: No secrets stored in Git or Helm values
- **Audit Trail**: All secret access logged in Azure Key Vault
- **Rotation**: Secrets can be rotated in Key Vault without application changes

### Configuration
```yaml
externalSecrets:
  enabled: true
  secretStore:
    name: azure-keyvault-secret-store
    spec:
      provider:
        azurekv:
          vaultUrl: "https://your-keyvault.vault.azure.net/"
          tenantId: "your-tenant-id"
          authType: WorkloadIdentity
```

## 📋 Scripts Overview

### Deployment Scripts
- **`deploy-infra.sh`**: Deploy Azure infrastructure using Terraform
- **`deploy-helm.sh`**: Deploy the Verify application using Helm charts
- **`validate.sh`**: General deployment validation

### Utility Scripts
- **`validate-config.sh`**: Configuration validation for different deployment styles
- **`generate-helm-values-updated.sh`**: Generate Helm values from Terraform outputs
- **`kubectl-invoke.sh`**: Helper script for private cluster access

### Common Workflows
```bash
# Full deployment
./providers/azure/scripts/deploy-infra.sh
./providers/azure/scripts/deploy-helm.sh
./scripts/validate.sh

# Update application
./providers/azure/scripts/deploy-helm.sh \
  --upgrade-only \
  --cluster-name <aks-name> \
  --resource-group <aks-resource-group> \
  --chart-package helm-charts/verify/verify-<version>.tgz \
  --values my-values.yaml
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with multiple providers
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For support and questions:
- Create an issue in this repository
- Contact the development team
- Check the troubleshooting section above
- Review provider-specific documentation

---

**Note**: This infrastructure is designed for production use with proper security configurations. Ensure you review and customize the security settings according to your organization's requirements and cloud provider best practices.
