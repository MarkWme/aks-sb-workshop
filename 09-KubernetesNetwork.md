# Azure Kubernetes Service Secure Baseline Workshop

## Part 9: Kubernetes Networking

In this section, we cover the Kubernetes specific networking components.

## Concepts

One of the first decisions that needs to be made when considering Kubernetes networking, is which networking model to use - kubenet or Azure CNI. There are a number of factors to consider when making this decision, but for the secure baseline we want to implement Azure Network Policy and this requires Azure CNI.

Azure Network Policy is a feature that allows you to control the traffic flow between pods, effectively a firewall running inside the Kubernetes cluster. You can choose to allow or deny traffic based on settings such as assigned labels, namespaces or ports. This makes it possible to configure applications running in the cluster to only communicate with the specific endpoints that they need to.

Ingress, the traffic coming into our cluster, is being routed through an Azure Application Gateway with Web Application Firewall (WAF) enabled. The Application Gateway has a public IP address to accept external HTTPS traffic. It acts as a TLS termination point, so it will decrypt the traffic and then process the WAF rules to look for potential issues. If there are no problems, the traffic is then encrypted again and forwarded to the Ingress controller running in the AKS cluster.

You can choose from a number of products to implement Ingress in Kubernetes. Traefik (simply pronounced "traffic") has been selected for the secure baseline implementation simply to show how third party products can be integrated with Azure services. In this case, we're using Pod Identity to allow Traefik to use a Managed Identity to access Azure Key Vault.

Application Gateway and Traefik both require access to certificates. A TLS certificate is used by Application Gateway to terminate incoming traffic and then a second certificate is used to re-encrypt traffic before it is sent on to the Ingress controller. The certificates are securely stored in Azure Key Vault and private link is used to restrict network access to Key Vault, preventing access from external networks and subnets other than those specifically enabled.

Traefik uses one of those certificates to terminate traffic incoming from Application Gateway. It uses the Azure Key Vault provider for Secrets Store CSI driver to provide access to a managed identity, which is then used to access Azure Key Vault and to retrieve the certificate.


How app gateway is being used as ingress, App Gateway WAF. Egress is through Firewall.

Traefik

Certificates in use, their purpose and storage in Key Vault

Private Link

---

## Exercises

Let's take a look at the flow of a network packet coming from the public Internet.



---

## Summary

---

## References

**Network concepts for applications in Azure Kubernetes Service**  
[https://docs.microsoft.com/en-us/azure/aks/concepts-network](https://docs.microsoft.com/en-us/azure/aks/concepts-network)

**Traefik**  
[https://traefik.io](https://traefik.io)
    - Azure CNI, network address space design
    - Azure Network Policy
    - App Gateway, Traefik Ingress, Certificates