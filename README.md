# AKS Secure Baseline Workshop
This workshop is designed to help you understand the Azure Kubernetes Service Secure Baseline. The secure baseline is a reference implementation which demonstrates the recommended starting infrastructure architecture for a general purpose AKS cluster.

Whilst the AKS Secure Baseline [repo](https://github.com/mspnp/aks-secure-baseline) provides a detailed step by step guide to deployment, this workshop is designed to firstly get the reference architecture deployed quickly and then deep dive into the features.

### **Contents**

0. [**Pre-requisites**](00-PreRequisites.md)
1. [**Deploying the network infrastructure**](01-NetworkDeployment.md)
2. [**Deploying and configuring the AKS cluster and supporting services.**](02-ClusterResourcesDeployment.md)

3. [**Authentication and Authorisation**](03-AuthNandAuthZ.md)
    - Accessing your cluster using Azure Active Directory for authentication
    - Understanding Kubernetes RBAC

4. [**Cluster Compute**](04-ClusterCompute.md)
    - Node pools
    - Nodes
    - Scaling, HPA and CA
    - System and user node pools
    - Memory reservation
    - Node maintenance, kured, node image updates

5. [**Identity Management**](05-IdentityManagement.md)
    - Managed Identity configuration

6. [**Container Registry**](06-ContainerRegistry.md)
    - Authentication
    - Importing public images
    - Geo replication

7. [**Cluster Configuration Management**](07-ClusterConfigMgmt.md)
    - GitOps / Flux
    - Investigate the YAML files
    - Namespaces
    - Components installed via GitOps - Pod Identity, Key Vault CSI

8. [**Azure Network Configuration**](08-AzureNetwork.md)
    - Hub and spoke network topology, peering
    - Subnets for App Gateway, Ingress and Cluster
    - Forced tunnel configuration
    - Network Security Group configuration
    - Azure Firewall configuration
    - Azure Load Balancer

9. [**Kubernetes Network Configuration**](09-KubernetesNetwork.md)
    - Azure CNI, network address space design
    - Azure Network Policy
    - App Gateway, Traefik Ingress, Certificates

10. [**Secret Management**](10-SecretManagement.md)
    - Key Vault configuration

11. [**Governance**](11-Governance.md)
    - Azure Policy
    - Understand the default policies that have been deployed

12. [**Application**](12-Application.md)
    - Deployment
    - Network policy
    - Traffic flow
    - PDB

13. [**Monitoring and Observability**](13-MonitoringandObservability.md)
    - Use Log Analytics to ...
