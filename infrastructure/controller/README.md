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
