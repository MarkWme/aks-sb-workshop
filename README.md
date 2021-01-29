# AKS Secure Baseline Workshop
This workshop is designed to help you understand the Azure Kubernetes Service Secure Baseline. The secure baseline is a reference implementation which demonstrates the recommended starting infrastructure architecture for a general purpose AKS cluster.

Whilst the AKS Secure Baseline [repo](https://github.com/mspnp/aks-secure-baseline) provides a detailed step by step guide to deployment, this workshop is designed to firstly get the reference architecture deployed quickly and then deep dive into the features.

### **Contents**

1. Deploying the network infrastructure
2. Deploying and configuring the AKS cluster and supporting services.
3. Deep dive - Network topology
    - Firewall
    - Load balancer
    - App Gateway / WAF
4. Deep dive - Container registry
5. Deep dive - Key Vault
6. Deep dive - Kubernetes cluster configuration
    - Security
    - Namespaces
    - GitOps
-----
### Network topology
- Hub with subnets for Azure Firewall, on-prem gateway and Bastion
- Spoke with subnets for App Gateway, Ingress and Cluster nodes
- IP address space design
### **kured vs node image updates**

### Container Registry
- Cluster auth, importing public images, policy to enforce authorised registries
- Geo replication

### Cluster Compute
- System and user node pools
- Scaling? HPA, CA
- BCP, node counts, pod availability, requests, limits
- AZ's and multi region

### Security (Authentication and Authorisation)
- AAD integration (kubectl)
- SP and MI
- K8s RBAC
- AAD RBAC
- Pod Identity

### Ingress
- ILB (AKS managed)
- ALB in a dedicated subnet for Ingress
- Traefik, config and routing
- App Gateway
- Cert config

- Secure network flow, traffic flow in and out of the cluster, firewall config

- Secret management, Key Vault and CSI driver

- Azure Policy
