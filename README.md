# deployment
# AWS EKS Access Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [AWS SSO Configuration](#aws-sso-configuration)
3. [EKS Access Steps](#eks-access-steps)
4. [Troubleshooting](#troubleshooting)
5. [Best Practices](#best-practices)c
6. [Additional Commands](#additional-commands)

## Prerequisites

Required tools and installation commands:

# macOS Installation Guide using Homebrew

## Install Homebrew (if not installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Install All Required Tools
Single command to install everything:
```bash
brew install awscli kubectl helm skaffold
```

Or install individually:
```bash
# AWS CLI
brew install awscli

# kubectl
brew install kubectl

# Helm
brew install helm

# Skaffold
brew install skaffold
```

## Verify Installations
```bash
# Check all versions
aws --version
kubectl version --client
helm version
skaffold version
```

## Update Tools
```bash
# Update Homebrew first
brew update

# Upgrade all tools at once
brew upgrade

# Or upgrade individually
brew upgrade awscli
brew upgrade kubectl
brew upgrade helm
brew upgrade skaffold
```

## Troubleshooting

### Fix Homebrew Permissions
```bash
sudo chown -R $(whoami) /usr/local/bin /usr/local/share
```

### Reset Homebrew
```bash
brew doctor
brew cleanup
```

### Common Commands
```bash
# Check if package is installed
brew list | grep aws

# Check package info
brew info kubectl

# Remove a package
brew uninstall awscli

# Clean up old versions
brew cleanup
```
```
### Required AWS IAM Permissions
```
- eks:DescribeCluster
- eks:ListClusters
- sts:AssumeRole
- sso:*
```

## AWS SSO Configuration

### 1. Get AWS SSO Access Token
1. Visit [Gobblecube AWS SSO Portal](https://gobblecube.awsapps.com/start/#/?tab=accounts)
2. Login with your credentials
3. Select the appropriate account and role
4. Note: Tokens typically expire after 8 hours

### 2. Configure AWS SSO
```bash
aws configure sso
```
You will need:
- SSO Start URL: https://gobblecube.awsapps.com/start
- SSO Region: ap-south-1
- Default Region: ap-south-1
- Default Output Format: json
- CLI Profile Name: <your-profile-name>

## EKS Access Steps

### 1. AWS SSO Login
```bash
aws sso login --profile <your-profile-name>
```

### 2. Set AWS Profile
```bash
export AWS_PROFILE=<your-profile-name>
```

### 3. Verify Identity
```bash
aws sts get-caller-identity
```

### 4. Update Kubeconfig
```bash
aws eks update-kubeconfig \
  --name <cluster-name> \
  --region <region> \
  --role-arn <your-role-arn>
```

### 5. Test Access
```bash
kubectl get ns
```

## Troubleshooting

### Common Issues

1. **Anonymous User Error**
```json
{
  "kind": "Status",
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "code": 403
}
```
**Solution:**
- Check AWS SSO session
- Re-run AWS SSO login
- Verify AWS_PROFILE is set

2. **Expired Token**
```
An error occurred (ExpiredToken) when calling the AssumeRole operation
```
**Solution:**
- Re-run AWS SSO login
- Check system time synchronization
- Verify token duration settings

3. **Missing SSO Configuration**
```
Missing required SSO configuration values: sso_start_url, sso_region
```

### Quick Fixes

1. **Reset Kubeconfig**
```bash
# Backup existing config
cp ~/.kube/config ~/.kube/config.bak

# Create new config
aws eks update-kubeconfig --name <cluster-name> --region <region>
```

2. **Refresh AWS Credentials**
```bash
aws sso login --profile <your-profile-name>
```

3. **Verify Context**
```bash
kubectl config current-context
kubectl config get-contexts
```

## Best Practices

1. **Session Management**
   - Keep track of SSO session duration
   - Re-authenticate before session expiry
   - Use environment variables when needed

2. **Configuration Backup**
   - Regularly backup kubeconfig
   - Document custom settings
   - Maintain profile information

3. **Security**
   - Don't share credentials
   - Use appropriate IAM roles
   - Regular access review

## Additional Commands

### Kubectl Commands
```bash
# View all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# View current configuration
kubectl config view

# Test specific namespace access
kubectl get pods -n <namespace>
```

### AWS CLI Commands
```bash
# View SSO session info
aws sts get-caller-identity

# List available clusters
aws eks list-clusters

# Describe cluster
aws eks describe-cluster --name <cluster-name>
```

### Important Environment Variables
```bash
AWS_PROFILE=<your-profile-name>
AWS_DEFAULT_REGION=<your-region>
KUBECONFIG=~/.kube/config
```

## Notes
- Replace `<your-profile-name>` with your AWS SSO profile name
- Replace `<region>` with your AWS region
- Replace `<cluster-name>` with your EKS cluster name
- Replace `<your-role-arn>` with your IAM role ARN
- Replace `<namespace>` with your Kubernetes namespace

## Deployment Process

### 1. Update Service Tag
1. Navigate to your service's values file:
```bash
cd components/<project>/<service>/gobblecube-production/perf-mark/values.yml
```

2. Update the image tag with the latest commit ID:
```yaml
image:
  repository: gobblecube/<service-name>
  tag: '<commit-id>'  # Update this value
```

### 2. Deploy Service
1. Navigate to the production cluster directory:
```bash
cd clusters/gobblecube-production
```

2. View available deployment commands:
```bash
make
# This will show all available make commands
```

3. Deploy your service:
```bash
# For specific service deployment
make deploy-perf-<service-name>

# For all services in a project
make deploy-perf-<project-name>-all
```

### Common Deployment Commands
```bash
# Deploy specific service
make deploy-perf-monet-backend

# Deploy all services for a project
make deploy-perf-monet-all

# Deploy specific worker
make deploy-perf-monet-worker-scheduler
```

### Deployment Verification
```bash
# Check pod status
kubectl get pods -n <namespace>

# Check logs
kubectl logs -f <pod-name> -n <namespace>

# Check deployment status
kubectl get deployments -n <namespace>
```

## Troubleshooting Deployments

### Common Deployment Issues

1. **Image Pull Error**
   - Verify commit ID exists in repository
   - Check image repository access
   - Verify regcred secret exists

2. **Configuration Error**
   - Verify values.yml changes
   - Check for syntax errors
   - Verify environment variables

3. **Access Issues**
   - Refresh AWS SSO token
   - Verify namespace access
   - Check IAM roles

### Quick Deployment Fixes

1. **Token Expired During Deployment**
```bash
# Refresh AWS SSO token
aws sso login --profile <your-profile-name>

# Retry deployment
make deploy-perf-<service-name>
```

2. **Wrong Context**
```bash
# Verify context
kubectl config current-context

# Switch if needed
kubectl config use-context <correct-context>
```
