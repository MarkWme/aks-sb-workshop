# AKS Secure Baseline Workshop
This workshop is designed to help you understand the Azure Kubernetes Service Secure Baseline. The secure baseline is a reference implementation which demonstrates the recommended starting infrastructure architecture for a general purpose AKS cluster.

Whilst the AKS Secure Baseline [repo](https://github.com/mspnp/aks-secure-baseline) provides a detailed step by step guide to deployment, this workshop is designed to firstly get the reference architecture deployed quickly and then deep dive into the features.

### **Contents**

0. Pre-requisites
1. Deploying the network infrastructure
2. Deploying and configuring the AKS cluster and supporting services.

3. Authentication and Authorisation
    - Azure AD integration, using kubectl to access the cluster using AAD auth
    - How Azure AD auth works, kubelogin
    - Azure RBAC and K8s RBAC integration

4. Cluster Compute
    - Node spec
    - Scaling, HPA and CA
    - System and user node pools
    - Memory reservation
    - Node maintenance, kured, node image updates

5. Identity Management
    - Managed Identity configuration

6. Container Registry
    - Authentication
    - Importing public images
    - Geo replication

7. Cluster Configuration Management
    - GitOps / Flux
    - Investigate the YAML files
    - Namespaces
    - Components installed via GitOps - Pod Identity, Key Vault CSI

8. Azure Network Configuration
    - Hub and spoke network topology, peering
    - Subnets for App Gateway, Ingress and Cluster
    - Forced tunnel configuration
    - Network Security Group configuration
    - Azure Firewall configuration
    - Azure Load Balancer

9. Kubernetes Network Configuration
    - Azure CNI, network address space design
    - Azure Network Policy
    - App Gateway, Traefik Ingress, Certificates

10. Secret Management
    - Key Vault configuration

11. Governance
    - Azure Policy
    - Understand the default policies that have been deployed

12. Application
    - Deployment
    - Network policy
    - Traffic flow

13. Monitoring and Observability
    - Use Log Analytics to ...
