# Automated VM Provisioning on Proxmox VE for Talos Kubernetes Cluster

## 🚀 Project Overview
This project demonstrates an Infrastructure as Code (IaC) approach to provisioning and managing virtual machine (VM) infrastructure for a Kubernetes cluster on Proxmox VE. Using Terraform, this solution automates the creation of a specified number of VMs for Control Plane and Worker nodes, designed and optimized specifically to run a Talos Linux-based Kubernetes cluster.

## 🎯 Problem & Solution

### The Problem
Manually setting up VMs for a Kubernetes cluster on bare-metal or on-premise hypervisors is a repetitive and error-prone process. This can lead to configuration drift and inconsistencies, making it difficult to scale and maintain a production-ready environment.

### The Solution
This project provides a fully automated, repeatable, and declarative solution using Terraform. By codifying the VM infrastructure, we achieve:

- **⚡ Automation**: Reduces VM provisioning time from hours to minutes
- **🎯 Consistency**: Ensures all VMs are created with identical, production-ready configurations
- **📈 Scalability**: Easily scale the number of VMs by changing a single variable
- **🔧 Optimization**: Configures VMs with hardware acceleration and optimized settings for high-performance workloads

## 🏗️ Architecture & Tech Stack

### Key Technologies
- **🏗️ IaC (Infrastructure as Code)**: Terraform for declarative infrastructure management
- **🖥️ Hypervisor**: Proxmox VE for on-premise virtualization  
- **🛡️ VM OS**: Optimized for secure and minimal OS like Talos Linux

### What Gets Created
- **N Control Plane nodes** (default: 3) - Master nodes that manage the cluster
- **M Worker nodes** (default: 3) - Worker nodes that run your applications

Each VM is configured with customizable CPU cores, memory, and disk size to fit your hardware requirements.

### Role in the Full Cluster Deployment Pipeline
This Terraform project is Stage 1: Infrastructure Provisioning. The complete setup of a functional Talos Kubernetes cluster involves two distinct stages:

Stage 1 (This Project): terraform apply - Provisions the raw VMs on Proxmox.

Stage 2 (Cluster Bootstrap): talosctl apply-config, talosctl bootstrap - Configures Talos OS on the VMs and bootstraps the Kubernetes cluster.

This separation of concerns is a best practice, allowing you to manage infrastructure and cluster configuration independently.

## 🚀 Getting Started

### 📋 Prerequisites
- **Proxmox VE server** with root access
- **Talos Linux ISO** uploaded to Proxmox storage (e.g., to local:iso/talos-amd64.iso)
- **Terraform** installed on your local machine
- Sufficient hardware resources for the planned VMs (required for the next stage after VM creation)

### 🏃 Quick Start

```bash
# 1. Clone repository
git clone <repo-url>
cd talos-proxmox-terraform

# 2. Install Terraform (if not installed)
chmod +x terraform-install.sh
./terraform-install.sh

# 3. Deploy VMs with interactive script
chmod +x vms-creation.sh
./vms-creation.sh
```

**Two configuration options:**
- **Option 1**: Use `terraform.tfvars` file (recommended for repeated deployments)
- **Option 2**: Enter settings manually during script execution (quick start)

### 📁 Project Structure

```
talos-proxmox-terraform/
├── main.tf                    # VM resource definitions
├── variables.tf               # Variable declarations with defaults
├── outputs.tf                 # Deployment information display
├── terraform.tfvars.example   # Configuration template
├── terraform-install.sh       # Terraform installation script
├── vms-creation.sh            # Interactive deployment script
├── .gitignore                 # Git ignore rules
└── README.md                  # This documentation
```

### 📄 File Descriptions

| File | Purpose | Modify? |
|------|---------|---------|
| **`variables.tf`** | Variable declarations with sensible defaults | Usually no |
| **`terraform.tfvars.example`** | Configuration template | Copy to `terraform.tfvars` and edit |
| **`main.tf`** | VM resource definitions | Usually no |
| **`outputs.tf`** | Deployment results display | Usually no |
| **`vms-creation.sh`** | Interactive deployment script | Usually no |

## ⚙️ Configuration Options

### 🎛️ VM Resources (configurable via terraform.tfvars)

| Setting | Control Plane Default | Worker Default | Description |
|---------|----------------------|----------------|-------------|
| **CPU Cores** | 2 | 2 | CPU cores per VM |
| **Memory** | 2048 MB | 2048 MB | RAM per VM |
| **Disk** | 20 GB | 32 GB | Storage per VM |
| **Count** | 3 | 3 | Number of VMs |

### 🌐 Infrastructure Settings

| Setting | Default | Description |
|---------|---------|-------------|
| **Storage** | `local-lvm` | Proxmox storage for VM disks |
| **Network Bridge** | `vmbr0` | Network bridge for VMs |
| **ISO Path** | `local:iso/metal-amd64.iso` | Path to Talos ISO |

## 🔧 Alternative Deployment Methods

### Method 1: Interactive Script (Recommended)
```bash
chmod +x vms-creation.sh
./vms-creation.sh
# Choose option 1 (settings from file) or 2 (manual input)
```

### Method 2: Direct Terraform Commands
```bash
# 1. Configure settings
cp terraform.tfvars.example terraform.tfvars
nano terraform.tfvars  # Edit with your settings

# 2. Set password
export TF_VAR_proxmox_password="your-proxmox-password"

# 3. Deploy
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```

### Method 3: Manual Configuration (No Files)
```bash
# Set all variables via environment
export TF_VAR_proxmox_endpoint="https://192.168.1.100:8006/api2/json"
export TF_VAR_proxmox_username="root@pam"
export TF_VAR_proxmox_password="your-password"
export TF_VAR_target_node="pve"

# Deploy with defaults
terraform init
terraform apply
```

## 🔍 After Deployment

After successful deployment, you'll see:
- ✅ **VM Information**: Names, IDs, resource allocation
- 📊 **Summary Statistics**: Total resources used  
- 📋 **Next Steps**: How to configure Talos and create Kubernetes cluster

## Next Steps: Configure Talos and Bootstrapping the Kubernetes Cluster

1. **Access VM Consoles**: Use Proxmox web interface
2. **Install Talosctl**: Download from [Talos Releases](https://github.com/siderolabs/talos/releases)
3. **Configure Cluster**: Follow [Talos Documentation](https://www.talos.dev/v1.10/kubernetes-guides/configuration/)
4. **Bootstrap Cluster**: Use talosctl to create your Kubernetes cluster

Here is a high-level overview with commands:

### Generate machine secrets and configs for a cluster with a virtual IP (VIP)
talosctl gen secrets -o secrets.yaml
talosctl gen config my-cluster https://my-cluster-vip:6443 \
  --with-secrets secrets.yaml \
  --output-types controlplane,worker
This creates 'controlplane.yaml' and 'worker.yaml'

### Apply the controlplane config to each master node
for node in 192.168.1.11 192.168.1.12 192.168.1.13; do
  talosctl apply-config --insecure --nodes $node --file controlplane.yaml
done
### Apply the worker config to each worker node
for node in 192.168.1.21 192.168.1.22 192.168.1.23; do
  talosctl apply-config --insecure --nodes $node --file worker.yaml
done

### Bootstrap the Cluster:
talosctl config endpoint 192.168.1.11 192.168.1.12 192.168.1.13
talosctl bootstrap --nodes 192.168.1.11

## 🐛 Troubleshooting & FAQ

### Common Issues

**❌ "Connection to Proxmox fails"**
```bash
# Test connection manually
curl -k https://your-proxmox:8006/api2/json/version
# Verify: endpoint URL, user permissions, password
```

**❌ "ISO not found"** 
```bash
# Check ISO exists on Proxmox
ssh root@proxmox-server "ls /var/lib/vz/template/iso/"
# Download latest Talos ISO
wget https://github.com/siderolabs/talos/releases/latest/download/metal-amd64.iso
```

**❌ "Insufficient resources"**
- Check Proxmox resources: `htop`, `free -h`, `df -h`
- Reduce VM counts or resources in configuration

**🔍 Debug Mode**
```bash
# Enable Terraform debug logging
export TF_LOG=DEBUG
terraform apply
```

## 🛠️ For reference - Terraform Management Commands
### Terraform

```bash
# View current state
terraform show

# Update deployment (after modifying variables)
terraform plan
terraform apply

# Destroy all VMs  
terraform destroy

# View outputs again
terraform output
```

### Proxmox

```
# Stop all VMs with name contains 'talos'

qm list | grep 'talos' | awk '{print $1}' | xargs -I {} qm stop {}

# Delete all VMs with name contains 'talos'
qm list | grep 'talos' | awk '{print $1}' | xargs -I {} qm destroy {} --purge

```

## ✅ Tested Environment & Validation

This configuration has been fully tested and validated in my Home Lab on lightweight server #3, demonstrating efficiency and portability:

### Intel N150 Home Lab (4 cores, 16GB RAM)
- **Control Plane**: 3 VMs × (2 CPU + 2GB RAM) = 6 CPU + 6GB RAM
- **Workers**: 3 VMs × (2 CPU + 2GB RAM) = 6 CPU + 6GB RAM  
- **Total Usage**: 12 CPU cores + 12GB RAM (leaves 4GB for Proxmox host)
- **Result**: ✅ Successfully deployed and running

## 🔗 Connecting to Other Projects

This project serves as the foundational infrastructure layer for:

- **🚢 Kubernetes Deployment**: Bootstrap full K8s cluster with talosctl
- **📦 Containerization**: Run containerized applications or Docker Swarm
- **🤖 Edge AI/IoT**: Deploy Edge AI workloads on the configured VMs
- **🔄 CI/CD Pipelines**: Use as target environment for deployments

## 👨‍💻 About the Author

This project showcases my expertise in Platform Engineering, DevOps, MLOps, AgentOps and Infrastructure as Code (IaC). Part of a larger portfolio demonstrating modern infrastructure automation practices of Technical Project Manager.

**Author**: Sergei Lavrentev  
**LinkedIn**: [www.linkedin.com/in/slavrent](https://www.linkedin.com/in/slavrent)

## 📄 License

MIT License - Feel free to use, modify, and distribute.

## 🤝 Contributing

Contributions welcome! Please feel free to submit issues or pull requests.

---

**🎯 Ready to deploy your Talos Kubernetes cluster? Start with `./vms-creation.sh`**
