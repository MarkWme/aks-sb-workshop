# Azure Kubernetes Service Secure Baseline Workshop

## Part 6: Container Registry

## Concepts

Azure Container Registry allows you to build, store and manage container images in a private registry.

Whilst it seems obvious that you would want to store any container images for your company's in-house developed applications in a private registry, it's also a good idea to do this for publicly available images. Whilst Docker Hub, for example, contains up to date images for a number of popular vendor's products, there are a number of reasons why you wouldn't want to rely on these images.

- **Security** - Whilst most vendors do a good job of ensuring that their images are safe and free from malware or security flaws, there have been incidents where, either maliciously or innocently, images with serious security problems have been posted to Docker Hub.
- **Reliability** - Docker Hub is, for the most part, a free service and carries no guarantees for uptime. Docker Hub does have outages ([Docker System Status History](https://status.docker.com/pages/history/533c6539221ae15e3f000031)) and this will impact your ability to build or deploy container images.
- **Service Limits** - Docker recently introduced policies to try to limit potentially excessive resource usage in Docker Hub. Anonymous pull requests from the same IP are now limited to 100 per 6 hours, whilst pull requests from free, signed in, Docker accounts are limited to 200 per 6 hours. There are also plans to force expiration of inactive images.

A sensible approach is to keep *all* of your container images in a private container registry. This includes images used for your own applications, any third-party applications and any images you rely on as a base to construct other images. This will protect you from service limits and reliability issues. You should combine this with security tooling to ensure that the images stored in your registry are clean.

Azure Container Registry features a Geo-Replication feature, which will automatically replicate container images to other Azure regions and then serve pull requests for those images from the nearest location.

Access to the container images in Azure Container Registry can be secured through Azure AD identities and role based access control. In the secure baseline, access to Azure Container Registry from the AKS cluster is granted using a managed identity.

The secure baseline also enables private link for Azure Container Registry, which establishes a private endpoint on the subnet where the AKS nodes reside and disables access from public networks. This means that you can only access the container registry from the AKS cluster nodes.

---

## Exercises

Let's import a third party container image into Azure Container Registry so that we have our own secure copy. In the secure baseline, we're making use of the Flux GitOps tooling, so let's take our own copy of that.

First, find the name of your container registry
```
az acr list -o table
```

The result will be something like below

>```
>NAME                 RESOURCE GROUP    LOCATION    SKU      LOGIN SERVER                    
>-------------------  ----------------  ----------  -------  ------------------------------  
>acraksoqchal7ga453i  akssb-cluster     westeurope  Premium  acraksoqchal7ga453i.azurecr.io  
>```

Next, let's import the Flux image from Docker Hub to Azure Container Registry

```
az acr import --source docker.io/fluxcd/flux:1.21.1 -n <insert ACR name here>
```

You will now have a copy of the Flux container image in your ACR instance, however, as we have enabled private link you won't be able to confirm this easily.

Try to view the repositories in your container registry

```
az acr repository list -n <insert ACR name here>
```

The result will be an error, usually something indicating access is not allowed! Private link is preventing access from outside of the AKS subnet.

---

## Challenges

- Geo-replication is enabled for your Azure Container Registry instance. Can you find where this is configured? Which Azure region is setup as a replica?
- Temporarily enable public access to your Azure Container Registry instance. Confirm that the command above to view the repositories now works.

---

## Summary

Azure Container Registry provides a number of ways to ensure your container images are secure.

---

## References

**Understanding Docker Hub Rate Limiting**  
[https://www.docker.com/increase-rate-limits](https://www.docker.com/increase-rate-limits)

**Resource Consumption Updates (Docker)**  
[https://www.docker.com/pricing/resource-consumption-updates](https://www.docker.com/pricing/resource-consumption-updates)