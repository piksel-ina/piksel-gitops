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
