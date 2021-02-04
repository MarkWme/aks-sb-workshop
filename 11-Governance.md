# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will explore ways that organizations can provide governance within their AKS Clusters.

---

## Part 11: Governance

## Concepts

An effective way to manage an AKS cluster is by enforcing governance through policies. Kubernetes implements policies through Open Policy Agent (OPA) Gatekeeper. For AKS, the policies are delivered through Azure Policy. Each policy is applied to all clusters in its scope. Azure Policy enforcement is ultimately handled by OPA Gatekeeper in the cluster, and all policy checks are logged. Policy changes are not immediately reflected in your cluster, expect to see some delay while syncing occurs.

The language for writing policies with OPA is called **rego**.

---

## Exercises

Let's explore AKS Policy (Azure Policy + OPA Gatekeeper) and some of the ways it can be used to govern AKS.

```bash
# Let's dig into what AKS Policy Looks Like inside of AKS
# Let's see which policies, ConstraintTemplates in AKS terminology, exist
kubectl get constrainttemplates,constraints
# Let's see which the constraints associated with the ConstraintTemplates
kubectl get constraint

# Let's dig deeper into one of the ConstraintTemplates (k8sazurecontainerallowedimages)
kubectl get constraints k8sazurecontainerallowedimages -o yaml | grep -i rego
kubectl get constraints k8sazurecontainerallowedimages -o yaml

# Let's see the constraints associated with k8sazurecontainerallowedimages
kubectl get k8sazurecontainerallowedimages
# Let's dig into one
kubectl get k8sazurecontainerallowedimages $(kubectl get k8sazurecontainerallowedimages -o jsonpath='{.items[0].metadata.name}') -o yaml | grep -i imageregex
```

**Challenge**
See if you can find the corresponding Azure Policy in the Portal.

**NOTE**
Feel free to explore some of the other policies (ConstraintTemplates).

---

## Summary

You should now have a better understanding of how Azure Policy and OPA Gatekeeper work together to help govern AKS Clusters.

---

## References

- [Open Policy Agent](https://github.com/open-policy-agent/opa)
- [Gatekeeper](https://github.com/open-policy-agent/gatekeeper)
- [AKS Baseline Policy Management](https://docs.microsoft.com/azure/architecture/reference-architectures/containers/aks/secure-baseline-aks#policy-management)

---
