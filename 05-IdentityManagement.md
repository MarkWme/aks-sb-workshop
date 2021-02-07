# Azure Kubernetes Service Secure Baseline Workshop

## Part 5: Identity Management

In this section, we'll examine the use of identities within the secure baseline.

## Concepts

Identities are required throughout Azure and within applications to authenticate access to various services. Azure has the concept of a Managed Identity, a wrapper around a service principal which automates the lifecycle of the identity, automatically creating it when needed, handling credential rotation and removing the identity when it is no longer required. Managed Identity removes the management overheads and some of the security risks of working with identities.

When you deploy an AKS cluster, it creates a new resource group into which the Azure resources that form your cluster, such as virtual machines, load balancers and IP addresses are deployed. In order to have permission to do that in your subscription, an identity is needed. In the secure baseline implementation, AKS is configured to use Managed Identities, as this removes the need to create and maintain a service principal.

You can use managed identities to allow access to Azure resources. In this scenario, managed identities are used to allow the AKS cluster to authenticate itself to an Azure Container Registry, allowing it to securely pull container images. They are also used by Azure Application Gateway to allow it to access a certificate stored in Azure Key Vault.

Another use case demonstrated here is the use of Pod Managed Identities. This is an add-on to AKS that allows applications running in pods to leverage managed identities for access to Azure services.

---

## Exercises

List the managed identities used in the secure baseline environment:

```
az identity list --query "[].{Name:name, ResourceGroup:resourceGroup}" -o table
```

>```
>Name                               ResourceGroup
>---------------------------------  ------------------------------
>podmi-ingress-controller           akssb-cluster
>mi-appgateway-frontend             akssb-cluster
>mi-aks-oqchal7ga453i-controlplane  akssb-cluster
>omsagent-aks-oqchal7ga453i         rg-aks-oqchal7ga453i-nodepools
>azurepolicy-aks-oqchal7ga453i      rg-aks-oqchal7ga453i-nodepools
>aks-oqchal7ga453i-agentpool        rg-aks-oqchal7ga453i-nodepools
>```

The names of the managed identities in your deployment might differ in some cases, but they should align to the following

Identity | Usage
---| ----
`<cluster name>-agentpool` | Primarily used for authentication with Azure Container Registry
`mi-<cluster-name>-controlplane` | Allows the AKS control plane to manipulate resources in your Azure subscription
`omsagent-<cluster name>` | Allows the metrics agent to publish metrics to Azure Monitor
`mi-appgateway-frontend` | Allows Application Gateway to access secrets in Azure Key Vault
`podmi-ingress-controller` | Used to allow the Traefik ingress controller to access secrets in Azure Key Vault
`azurepolicy-<cluster-name>` | Created by the Azure Policy add-on

---

## Summary

Managed Identities provide an easy way to allow applications and services to access resources without having to concern yourself with the lifecycle of identities.

---

## References

**Use managed identities in Azure Kubernetes Services**  
[https://docs.microsoft.com/en-us/azure/aks/use-managed-identity](https://docs.microsoft.com/en-us/azure/aks/use-managed-identity)

**Use Azure Active Directory pod-managed identities in Azure Kubernetes Service (Preview)**  
[https://docs.microsoft.com/en-us/azure/aks/use-azure-ad-pod-identity](https://docs.microsoft.com/en-us/azure/aks/use-azure-ad-pod-identity)