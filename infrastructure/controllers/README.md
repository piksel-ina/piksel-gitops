## cert-manager Setup

This configuration deploys cert-manager using Flux CD (GitOps) to automate TLS certificate management in the Kubernetes cluster.

### About cert-manager

cert-manager is a Kubernetes add-on that automatically manages and issues TLS certificates from various certificate authorities (CAs) like Let's Encrypt. It eliminates the need for manual certificate management and ensures automatic renewal before expiration.

### Configuration Details

1. **Components**

   - **Namespace**: Dedicated `cert-manager` namespace for resource isolation
   - **HelmRepository**: Points to official Jetstack chart repository
   - **HelmRelease**: Manages the cert-manager deployment via Flux CD

2. **Key Settings**

   - **version**: "1.17.x" # Uses latest patch version in 1.17 series
   - **Update intervals**:
     - interval: 30m # Flux reconciliation interval
     - chart.interval: 10m # Chart update check interval
     - repo.interval: 24h # Repository update check interval

3. **Installation**
   - **installCRDs**: true # Installs required Custom Resource Definitions
   - **retries**: 3 # Installation retry attempts

### Default Behavior

This configuration uses **default values** from the cert-manager Helm chart, which include:

- Single replica deployment
- Standard resource limits (100m CPU, 128Mi memory)
- Security contexts with non-root user
- Log level 2 (standard verbosity)
- Service account auto-creation

### This Enables:

- **Automatic Certificate Management**

  - Issuance: Automatically requests certificates from CAs
  - Renewal: Renews certificates before expiration (typically 30 days before)
  - Storage: Stores certificates as Kubernetes secrets
  - Monitoring: Watches for certificate-related resources

- **GitOps Integration**
  - Declarative: Managed through Git commits
  - Self-healing: Automatically maintains desired state
  - Updates: Handles chart updates automatically

### Verification

After deployment, verify cert-manager is working:

```bash
# Check cert-manager pods
kubectl get pods -n cert-manager

# Check CRDs are installed
kubectl get crd | grep cert-manager

# Check cert-manager logs
kubectl logs -n cert-manager deployment/cert-manager
```

### References

- [cert-manager Documentation](https://cert-manager.io/docs/)
- [Jetstack Helm Charts](https://artifacthub.io/packages/helm/cert-manager/cert-manager)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)

## NGINX Ingress Controller Setup

This configuration deploys the NGINX Ingress Controller using Flux CD (GitOps) to manage HTTP/HTTPS traffic routing into the Kubernetes cluster.

### About NGINX Ingress Controller

The NGINX Ingress Controller is a Kubernetes ingress controller that uses NGINX as a reverse proxy and load balancer. It manages external access to services in a cluster, typically HTTP/HTTPS, and provides features like SSL termination, name-based virtual hosting, and load balancing.

### Configuration Details

1. **Components**

   - **Namespace**: Dedicated `ingress-nginx` namespace for resource isolation
   - **HelmRepository**: Points to official Kubernetes NGINX Ingress chart repository
   - **HelmRelease**: Manages the NGINX Ingress Controller deployment via Flux CD

2. **Key Settings**

- **version**: "4.12.x" # Uses latest patch version in 4.12 series
- **Update intervals**:
  - interval: 30m # Flux reconciliation interval
  - chart.interval: 12h # Chart update check interval
  - repo.interval: 24h # Repository update check interval

3. **Service Configuration**

- **type**: "LoadBalancer" # Exposes controller via cloud load balancer
- Gets external IP address for cluster ingress traffic

4. **Security & Header Configuration**

- **allowSnippetAnnotations**: true # ⚠️ Allows custom NGINX config snippets (security risk)
- **compute-full-forwarded-for**: "true" # Computes complete forwarded-for header
- **forwarded-for-header**: "X-Forwarded-For" # Preserves client IP addresses
- **use-forwarded-headers**: "true" # Uses forwarded headers from upstream proxies
- **annotations-risk-level**: Critical # Acknowledges security risk of snippets

5. **Disabled Features**

- **admissionWebhooks**: false # Disables validation webhooks for ingress resources

### This Enables:

1. **Traffic Management**

- **Ingress Routing**: Routes external HTTP/HTTPS traffic to internal services
- **Load Balancing**: Distributes traffic across multiple service endpoints
- **SSL Termination**: Handles TLS/SSL certificate termination at the edge
- **Virtual Hosting**: Supports multiple domains/subdomains on single IP

2. **Advanced Features**

- **Custom Configuration**: Allows NGINX configuration snippets via annotations
- **Client IP Preservation**: Maintains original client IP through proxy chains
- **Health Checks**: Built-in health checking for backend services
- **Rate Limiting**: Can implement rate limiting and throttling

3. **GitOps Integration**

- **Declarative**: Managed through Git commits
- **Self-healing**: Automatically maintains desired state
- **Controlled Updates**: Automatically gets patch updates within 4.12.x series

### Security Considerations

⚠️ **Important Security Notes:**

- `allowSnippetAnnotations: true` allows arbitrary NGINX configuration injection
- This can be exploited for privilege escalation or data exfiltration
- Consider disabling in production or implementing strict RBAC controls
- `annotations-risk-level: Critical` acknowledges this risk

### Verification

After deployment, verify NGINX Ingress Controller is working:

```bash
# Check ingress controller pods
kubectl get pods -n ingress-nginx

# Check service and external IP
kubectl get svc -n ingress-nginx

# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Test with a sample ingress
kubectl get ingress --all-namespaces
```

### Production Recommendations

1. **Security Hardening**

```yaml
# Recommended production values
allowSnippetAnnotations: false # Disable snippets
admissionWebhooks:
  enabled: true # Enable validation
```

2. **Resource Limits**

```yaml
# Add resource constraints
controller:
  resources:
    requests:
      cpu: 100m
      memory: 90Mi
    limits:
      cpu: 200m
      memory: 256Mi
```

### References

- [NGINX Ingress Controller Documentation](https://kubernetes.github.io/ingress-nginx/)
- [Kubernetes Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- [Helm Chart Repository](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

## Flux Slack Notification

This configuration creates separate Slack notification providers for Flux CD to send GitOps deployment notifications to environment-specific Slack channels.

### About Flux Notification Provider

The Flux Notification Provider enables Flux to send alerts and status updates about GitOps operations (deployments, reconciliations, failures) to external systems like Slack, Discord, Microsoft Teams, or webhook endpoints.

### Configuration Details

**Environment Separation Strategy**

- **Multiple Providers**: Separate providers for each environment
- **Dedicated Channels**: Environment-specific Slack channels
- **Isolated Webhooks**: Different webhook URLs for better security and management
- **Namespace**: Both deployed in `flux-system` namespace
- **Provider Name**: "slack-dev" and "slack-prod"
- **Channel**:
  - "flux-piksel-dev-env" # Development notifications channel
  - "flux-piksel-prod-env" # Production notifications channel
- **Secret**:
  - "slack-webhook-dev" # Development-specific webhook URL
  - "slack-webhook-prod" # Production-specific webhook URL

### This Enables:

1. **Environment-Specific Notifications**

- **Isolated Channels**: Dev and prod notifications are completely separated
- **Targeted Audience**: Different teams can subscribe to relevant channels
- **Noise Reduction**: Production teams only see critical prod alerts
- **Security Isolation**: Separate webhooks prevent cross-environment access

2. **Flexible Alert Management**

- **Different Severity Levels**: Dev can be verbose, prod can be critical-only
- **Environment Context**: Clear identification of which environment is affected
- **Independent Configuration**: Each environment can have different notification rules

### Required Setup

1. **Slack Channels**
   Create the required Slack channels in your workspace:

   - `#flux-piksel-dev-env` - Development environment notifications
   - `#flux-piksel-prod-env` - Production environment notifications

2. **Create separate secrets for each environment**

- Should be done in Terraform Repo [piksel-infra](https://github.com/piksel-ina/piksel-infra)

### Verification

After deployment, verify both providers are ready:

```bash
# Check both providers status
kubectl get provider -n flux-system

# Check individual provider details
kubectl describe provider slack-dev -n flux-system
kubectl describe provider slack-prod -n flux-system

# Verify secrets exist
kubectl get secret slack-webhook-dev slack-webhook-prod -n flux-system
```

### References

- [Flux Notification Documentation](https://fluxcd.io/flux/components/notification/)
- [Slack Webhook Setup Guide](https://api.slack.com/messaging/webhooks)
- [Flux Alert Configuration](https://fluxcd.io/flux/components/notification/alert/)

## NVIDIA GPU Device Plugin

This configuration deploys the NVIDIA Kubernetes Device Plugin to enable GPU workloads in the cluster by exposing NVIDIA GPUs as schedulable resources to the Kubernetes scheduler.

### About NVIDIA Device Plugin

The NVIDIA Device Plugin for Kubernetes is a DaemonSet that allows you to automatically expose the number of GPUs on each node of your cluster and run GPU-enabled containers. It implements the Kubernetes device plugin framework to advertise NVIDIA GPUs to the kubelet.

### Configuration Details

**Deployment Strategy**

- **Namespace**: Dedicated `gpu-operator` namespace with privileged security context
- **Chart Source**: Official NVIDIA Helm repository (`https://nvidia.github.io/k8s-device-plugin`)
- **Version**: `0.17.x` (automatic patch updates within v0.17 series)
- **Node Targeting**: Only deploys on nodes with NVIDIA GPUs via Karpenter labels
- **Management**: Fully managed by Flux CD with GitOps principles

**Security Configuration**

```yaml
labels:
  pod-security.kubernetes.io/enforce: privileged # Required for GPU driver access
  pod-security.kubernetes.io/audit: baseline # Security monitoring
  pod-security.kubernetes.io/warn: baseline # Security warnings
```

**Node Selection Strategy**

```yaml
nodeSelector:
  karpenter.k8s.aws/instance-gpu-manufacturer: nvidia
```

### This Enables:

1. **GPU Resource Discovery**

- **Automatic Detection**: Discovers all NVIDIA GPUs on worker nodes
- **Resource Advertising**: Exposes GPUs as `nvidia.com/gpu` resources to Kubernetes scheduler
- **Health Monitoring**: Continuously monitors GPU health and availability
- **Hot-plug Support**: Automatically detects GPU changes without node restart

2. **Workload Scheduling**

- **Resource Requests**: Pods can request GPU resources using `nvidia.com/gpu: 1`
- **Exclusive Access**: Ensures GPU isolation between containers
- **Multi-GPU Support**: Supports workloads requiring multiple GPUs
- **Mixed Workloads**: Allows CPU and GPU workloads on same nodes

3. **AWS Integration**

- **Karpenter Compatibility**: Works seamlessly with Karpenter node provisioning
- **Instance Type Support**: Compatible with p3, p4, g4, g5, and other GPU instance types
- **Cost Optimization**: Only runs on nodes that actually have GPUs
- **Auto-scaling**: Supports dynamic GPU node scaling based on workload demands

### Supported GPU Workloads

- **Machine Learning**: TensorFlow, PyTorch, CUDA-based training and inference
- **AI/Deep Learning**: Model training, neural network inference
- **High-Performance Computing**: Scientific computing, simulations
- **Graphics Rendering**: GPU-accelerated rendering and visualization
- **Cryptocurrency**: Mining and blockchain operations
- **Video Processing**: Transcoding, streaming, computer vision

### Required Setup

1. **Infrastructure Prerequisites**

   - AWS EKS cluster with GPU-enabled worker nodes
   - Karpenter configured with GPU instance types
   - NVIDIA drivers installed on worker nodes (typically via AMI)

2. **Node Configuration**
   Ensure your Karpenter NodePool includes GPU instance types:
   ```yaml
   requirements:
     - key: karpenter.k8s.aws/instance-family
       operator: In
       values: ["p3", "p4", "g4dn", "g5"]
   ```

### Verification

After deployment, verify the device plugin is working:

```bash
# Check device plugin pods are running
kubectl get pods -n gpu-operator

# Verify GPUs are detected on nodes
kubectl describe nodes | grep -A 5 "Allocatable"
kubectl get nodes -o custom-columns="NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"

# Test with a simple GPU workload
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: gpu-test
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:12.0-runtime-ubuntu20.04
    command: ["nvidia-smi"]
    resources:
      limits:
        nvidia.com/gpu: 1
  restartPolicy: Never
EOF

# Check GPU test results
kubectl logs gpu-test
```

### References

- [NVIDIA Device Plugin Documentation](https://github.com/NVIDIA/k8s-device-plugin)
- [Kubernetes Device Plugin Framework](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/overview.html)
- [AWS EKS GPU Support](https://docs.aws.amazon.com/eks/latest/userguide/gpu-ami.html)
