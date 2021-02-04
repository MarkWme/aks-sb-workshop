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
kubectl get networkpolicies,netpol
kubectl get networkpolicies,netpol -A
```

**NOTE**
Try creating your own Network Policy to better understand how it works.

---

## Summary

You should now have a better understanding of how Network Policies can be used to help govern network traffic within, as well as in and out of the AKS Cluster.

---

## References

- [K8s Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Network Policy Viewer](https://orca.tufin.io/netpol/#)
- [AKS Baseline Network Policy](https://docs.microsoft.com/azure/architecture/reference-architectures/containers/aks/secure-baseline-aks#pod-to-pod-traffic)

---
