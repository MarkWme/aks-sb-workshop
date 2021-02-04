# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will explore the concept of GitOps using an open source tool called Flux which is deployed as part of the AKS Baseline.

---

## Part 7: Cluster Configuration Management

## Concepts

Instead of using an **imperative** approach like kubectl, we are going to explore a tool called **Flux** that automatically synchronizes cluster and repository changes. Kubernetes and AKS do not support that experience natively. To manage the workflow, such as release of a new version and validation of that version before deploying to production, consider a **GitOps** flow. Flux deploys an agent in the cluster to make sure that the state of the cluster is coordinated with configuration stored in a private Git repo.

---

## Exercises

Let's explore GitOps.

```bash
# First, let's check out existing namespaces.
kubectl get namespaces
```

**What do you notice?** There are a number of additional namespaces in addition to the default. How did those get there? Automation and **GitOps!**

```bash
# Let's Explore the Flux Agent
# First Check out cluster-baseline-settings namespace where Flux is deployed
kubectl -n cluster-baseline-settings get deploy,po,secrets
kubectl -n cluster-baseline-settings get deploy,po,secrets | grep -i flux

# Let's look at Flux Logs
kubectl -n cluster-baseline-settings logs -l app.kubernetes.io/name=flux
# OR if you want to see more output
kubectl -n cluster-baseline-settings logs -l app.kubernetes.io/name=flux --tail 1000
```

**What do you see in the logs?** There is a bunch of reconciliation going on. The agent is periodically checking the git repo for changes and comparing that to what is running in the cluster. If there are discrepancies then it's job is to reconcile them.

```bash
# Reconciliation in Action
# Which repo is Flux point too? 
kubectl -n cluster-baseline-settings get deploy flux -o yaml | grep -i git

# Delete AAD Pod Identity
kubectl -n cluster-baseline-settings get deploy,ds,po
kubectl -n cluster-baseline-settings get deploy aad-pod-identity-mic
kubectl -n cluster-baseline-settings delete deploy aad-pod-identity-mic
# Verify mic Deployment is gone
kubectl -n cluster-baseline-settings get deploy aad-pod-identity-mic
# Wait ~5mins (Flux Timeout Setting), what happens?
kubectl -n cluster-baseline-settings get deploy,ds,po
```

**Challenge**

- Can you find the log entry that shows where the **mic** deployment got added back?

**NOTE**

- Feel free to explore more and try adding your own Flux agent setup to the cluster.

---

## Summary

You should now have a better understanding of what GitOps is, the types of challenges it can solve, and what an implementation looks like.

---

## References

- [GitOps Definition from Weaveworks](https://www.weave.works/technologies/gitops/)
- [AKS Baseline Cluster CI/CD](https://docs.microsoft.com/azure/architecture/reference-architectures/containers/aks/secure-baseline-aks#cluster-cicd)

---
