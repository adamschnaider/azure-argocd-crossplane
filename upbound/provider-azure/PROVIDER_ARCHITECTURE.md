# Azure Provider Architecture

## Overview
This repository uses the **modular provider architecture** from Upbound for Crossplane Azure resources.

## Provider Strategy: Individual Service Providers

Instead of using the monolithic `provider-family-azure`, we deploy individual service-specific providers:

### Deployed Providers

| Provider | Package | Purpose | Resources |
|----------|---------|---------|-----------|
| `provider-azure-resources` | `provider-azure-resources:v1` | Resource management | ResourceGroup |
| `provider-azure-network` | `provider-azure-network:v1` | Networking resources | VirtualNetwork, Subnet |
| `provider-azure-storage` | `provider-azure-storage:v1` | Storage resources | Storage Account |
| `provider-azure-cognitiveservices` | `provider-azure-cognitiveservices:v1` | AI/ML services | Azure OpenAI, Cognitive Services |
| `provider-azure-containerservice` | `provider-azure-containerservice:v1` | Container orchestration | AKS (KubernetesCluster) |
| `provider-azure-compute` | `provider-azure-compute:v1` | Compute resources | Virtual Machines (reserved for future use) |
| `provider-azure-containerregistry` | `provider-azure-containerregistry:v1` | Container registries | ACR (reserved for future use) |

## Why Modular Providers?

### Advantages
1. **Smaller Footprint**: Each provider only includes APIs for its specific service
2. **Faster Updates**: Update only the providers you need
3. **Better Resource Management**: Less memory and CPU usage
4. **Granular Control**: Install only what you need
5. **Reduced Blast Radius**: Issues in one provider don't affect others

### API Patterns
Each provider follows the pattern: `<service>.azure.upbound.io/<version>`

Examples:
- `azure.upbound.io/v1beta1` - Core resources (ResourceGroup)
- `network.azure.upbound.io/v1beta2` - Network resources
- `storage.azure.upbound.io/v1beta1` - Storage resources
- `cognitiveservices.azure.upbound.io/v1beta1` - Cognitive services
- `containerservice.azure.upbound.io/v1beta1` - Container services

## Provider Configuration

All providers share the same ProviderConfig (`default`) which references the Azure credentials stored in the `azure-secret` within the `crossplane-system` namespace.

## Resources in Use

Current composition (`crossplane-composition-dev-env.yaml`) uses:

1. **ResourceGroup** (`azure.upbound.io/v1beta1`)
   - Provider: `provider-azure-resources`

2. **VirtualNetwork** (`network.azure.upbound.io/v1beta2`)
   - Provider: `provider-azure-network`

3. **Storage Account** (`storage.azure.upbound.io/v1beta1`)
   - Provider: `provider-azure-storage`

4. **Cognitive Services Account & Deployment** (`cognitiveservices.azure.upbound.io/v1beta1`)
   - Provider: `provider-azure-cognitiveservices`

5. **Kubernetes Cluster** (`containerservice.azure.upbound.io/v1beta1`)
   - Provider: `provider-azure-containerservice`

## Migration Note

Previously, `provider-family-azure` was included but was redundant as it provides the same APIs as the individual providers combined. It has been removed to:
- Eliminate duplication
- Reduce resource consumption
- Simplify maintenance
- Follow Upbound's recommended modular approach

## Adding New Providers

To add a new Azure service provider:

1. Create a new provider YAML in `upbound/provider-azure/provider/`:
```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-azure-<service>
spec:
  package: xpkg.upbound.io/upbound/provider-azure-<service>:v1
  packagePullPolicy: IfNotPresent
  revisionActivationPolicy: Automatic
  revisionHistoryLimit: 1
```

2. The provider will be automatically deployed by ArgoCD through the `crossplane-provider-azure` Application

3. Use resources from the provider in compositions with the API pattern: `<service>.azure.upbound.io/<version>`

## References

- [Upbound Provider Azure Family Documentation](https://marketplace.upbound.io/providers/upbound/provider-family-azure/)
- [Crossplane Provider Documentation](https://docs.crossplane.io/latest/concepts/providers/)
