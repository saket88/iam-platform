# Kustomize Setup - Step-by-Step Guide

## Prerequisites
- Lima VM with k0s is running (`limactl start k0s.yaml`)
- kubectl is installed on your host machine
- You have access to your Hetzner VPS

## Step 1: Extract Lima Kubeconfig
```bash
# Make sure your Lima VM is running
limactl start k0s.yaml

# Execute the extract kubeconfig script
bash extract_kubeconfig.sh

# Verify it worked
KUBECONFIG=~/.kube/config-lima-k0s kubectl get nodes
```

## Step 2: Create Project Structure
```bash
# Navigate to your workspace (or wherever you want the project)
cd /Volumes/T7/iam-platform  # Based on your Lima mount

# Create the directory structure
bash kustomize_structure.sh

# Verify structure was created
tree iam-platform-k8s/
```

## Step 3: Create Base Manifests
```bash
# Navigate to the project directory
cd iam-platform-k8s

# Create the base manifest files
# Copy the content from the "Base Kubernetes Manifests" artifact into these files:
cat > base/deployment.yaml << 'EOF'
# [Copy deployment.yaml content from artifact]
EOF

cat > base/service.yaml << 'EOF'
# [Copy service.yaml content from artifact]
EOF

cat > base/configmap.yaml << 'EOF'
# [Copy configmap.yaml content from artifact]
EOF
```

## Step 4: Create Environment Patches
```bash
# Still in iam-platform-k8s directory
# Create local environment patches
cat > overlays/local/replica-patch.yaml << 'EOF'
# [Copy local replica-patch.yaml content from artifact]
EOF

cat > overlays/local/resource-limits-patch.yaml << 'EOF'
# [Copy local resource-limits-patch.yaml content from artifact]
EOF

# Create hetzner environment patches
cat > overlays/hetzner/replica-patch.yaml << 'EOF'
# [Copy hetzner replica-patch.yaml content from artifact]
EOF

cat > overlays/hetzner/resource-limits-patch.yaml << 'EOF'
# [Copy hetzner resource-limits-patch.yaml content from artifact]
EOF
```

## Step 5: Create Deployment Scripts
```bash
# Create deployment scripts in the project root
cat > deploy-local.sh << 'EOF'
#!/bin/bash
echo "Deploying to Lima k0s cluster..."
KUBECONFIG=~/.kube/config-lima-k0s kubectl apply -k overlays/local/
EOF

cat > deploy-hetzner.sh << 'EOF'
#!/bin/bash
echo "Deploying to Hetzner cluster..."
KUBECONFIG=~/.kube/config-hetzner kubectl apply -k overlays/hetzner/
EOF

cat > preview-local.sh << 'EOF'
#!/bin/bash
echo "Previewing local deployment..."
KUBECONFIG=~/.kube/config-lima-k0s kubectl kustomize overlays/local/
EOF

cat > preview-hetzner.sh << 'EOF'
#!/bin/bash
echo "Previewing Hetzner deployment..."
KUBECONFIG=~/.kube/config-hetzner kubectl kustomize overlays/hetzner/
EOF

cat > switch-context.sh << 'EOF'
#!/bin/bash
case $1 in
  "local")
    export KUBECONFIG=~/.kube/config-lima-k0s
    echo "Switched to Lima k0s cluster"
    ;;
  "hetzner")
    export KUBECONFIG=~/.kube/config-hetzner
    echo "Switched to Hetzner cluster"
    ;;
  *)
    echo "Usage: $0 [local|hetzner]"
    exit 1
    ;;
esac
kubectl get nodes
EOF

# Make scripts executable
chmod +x *.sh
```

## Step 6: Test Local Deployment
```bash
# Preview what will be deployed (dry run)
bash preview-local.sh

# If everything looks good, deploy to Lima cluster
bash deploy-local.sh

# Verify deployment
KUBECONFIG=~/.kube/config-lima-k0s kubectl get pods -n iam-platform
```

## Step 7: Set Up Hetzner Cluster (When Ready)
```bash
# SSH to your Hetzner VPS and install k0s (similar to Lima setup)
ssh your-hetzner-vps

# On Hetzner VPS:
curl -sSLf https://get.k0sproject.io | bash
sudo mv k0s /usr/local/bin/
sudo k0s install controller --single
sudo k0s start

# Get kubeconfig from Hetzner
sudo k0s kubectl config view --raw > /tmp/kubeconfig

# Back on your local machine:
scp your-hetzner-vps:/tmp/kubeconfig ~/.kube/config-hetzner

# Test Hetzner connection
KUBECONFIG=~/.kube/config-hetzner kubectl get nodes
```

## Step 8: Deploy to Hetzner (When Ready)
```bash
# Preview Hetzner deployment
bash preview-hetzner.sh

# Deploy to Hetzner
bash deploy-hetzner.sh

# Verify deployment
KUBECONFIG=~/.kube/config-hetzner kubectl get pods -n iam-platform
```

## Step 9: Using the Switch Context Script
```bash
# Switch to local cluster
source switch-context.sh local

# Switch to Hetzner cluster
source switch-context.sh hetzner

# Note: Use 'source' to export the KUBECONFIG variable to your current shell
```

## Troubleshooting Commands
```bash
# Check if Lima VM is running
limactl list

# Check cluster status
KUBECONFIG=~/.kube/config-lima-k0s kubectl cluster-info

# Check namespace
KUBECONFIG=~/.kube/config-lima-k0s kubectl get namespaces

# Delete deployment if needed
KUBECONFIG=~/.kube/config-lima-k0s kubectl delete -k overlays/local/
```

## Important Notes
1. **Customize the manifests** - Update the base manifests with your actual application details
2. **Update image references** - Replace `your-registry/iam-platform:latest` with your actual image
3. **Test locally first** - Always test on Lima before deploying to Hetzner
4. **Use preview scripts** - Always preview before deploying to see what changes will be made

---

## File Information
- **Created**: $(date)
- **Purpose**: Step-by-step guide for setting up Kustomize with Lima k0s and Hetzner Cloud
- **Related**: [Complete Project Setup Script](complete_project_setup.sh)

## Quick Reference Commands

| Task | Command |
|------|---------|
| Start Lima VM | `limactl start k0s.yaml` |
| Extract kubeconfig | `bash extract_kubeconfig.sh` |
| Preview local | `bash preview-local.sh` |
| Deploy local | `bash deploy-local.sh` |
| Preview Hetzner | `bash preview-hetzner.sh` |
| Deploy Hetzner | `bash deploy-hetzner.sh` |
| Switch context | `source switch-context.sh [local\|hetzner]` |

## Execution Order Summary
1. **Steps 1-5**: Foundation setup (kubeconfig, structure, manifests, scripts)
2. **Step 6**: Local testing and validation
3. **Step 7**: Hetzner cluster preparation
4. **Step 8**: Production deployment
5. **Step 9**: Ongoing management with context switching

> **⚠️ Important**: Always use preview scripts before deploying to see exactly what changes will be made!

https://claude.ai/public/artifacts/b7602b65-a48b-477d-97bf-8373bfe0e3e0  as a reference artifact
