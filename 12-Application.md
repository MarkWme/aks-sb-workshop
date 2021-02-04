# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will explore the different ways that AKS can interact with the Application. The goal is to not spend time on the App, it is purposefully boring, but how it works within the AKS Baseline ecosystem.

---

## Part 12: Application

## Concepts

By default, a pod can accept traffic from any other pod in the cluster. **Kubernetes NetworkPolicy** is used to restrict network traffic between pods. Apply policies judiciously, otherwise you might have a situation where a critical network flow is blocked. Only allow specific communication paths, as needed, such as traffic between the ingress controller and workload.

**NOTE: Enable network policy when the cluster is provisioned because it can't be added later.**

---

## Exercises

Let's explore the NetworkPolicies that were created as part of **GitOps**.

```bash
# List NetworkPolicies
kubectl get networkpolicy,netpol
kubectl get networkpolicy,netpol -A

# Look at Application Network Policy
kubectl -n a0008 get networkpolicy allow-only-ingress-to-workload -o yaml

# Validate Ingress Network Policy Rule
kubectl run -it --rm centos --image=centos:8 -- /bin/bash
kubectl run -it --rm centos --image=centos:8 --requests 'cpu=250m,memory=512Mi' --limits 'cpu=250m,memory=512Mi' -- /bin/bash

# Hmmm, what can we do? Let's load image into Trusted ACR
az acr list -o table
ACR=<ACR_GOES_HERE>
az acr update --name $ACR --admin-enabled
az acr login --name $ACR
az acr update --name $ACR --public-network-enabled true
az acr login --name $ACR

# What can we do to resolve the constraints?
docker pull centos:8
docker tag centos:8 $ACR.azurecr.io/centos:8
docker push $ACR.azurecr.io/centos:8
az acr update --name $ACR --public-network-enabled false

# Let's try running the container again
kubectl run -it --rm centos --image="$ACR.azurecr.io/centos:8" --requests 'cpu=250m,memory=512Mi' --limits 'cpu=250m,memory=512Mi' -- /bin/bash

# Once we are inside of the container, let's try hitting some endpoints to see what we can, and cannot do.
# Public Endpoint Allowed via Firewall
curl http://security.ubuntu.com

# Application Service Endpoint
curl http://aspnetapp-service.a0008.svc.cluster.local/

# Now try by hitting Ingress Controller Endpoint
curl -k -H "Host: bicycle.contoso.com" https://10.240.4.4/

# Wait, why did that not work? Do we have the correct Host Header?
curl -k https://bu0001a0008-00.aks-ingress.contoso.com/

# Wait, that kind of worked, why did I get forbidden?

# Exit out of Pod
exit
```

**Challenge**

- Find out why the Pod in the default namespace could not access Ingress Controller endpoint. **Hint: The answer is in the documentation.**
- Create a deny all Network Policy in the default namespace, and then validate it by trying to curl **http://security.ubuntu.com**

**NOTE**

- Explore network policies some more to see what is, and is not possible.

---

## Summary

You should now have a better understanding of how Network Policies can be used to help govern network traffic within, as well as in and out of the AKS Cluster.

---

## References

- [K8s Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Network Policy Viewer](https://orca.tufin.io/netpol/#)
- [AKS Baseline Network Policy](https://docs.microsoft.com/azure/architecture/reference-architectures/containers/aks/secure-baseline-aks#pod-to-pod-traffic)

---
