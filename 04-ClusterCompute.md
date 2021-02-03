# Azure Kubernetes Service Secure Baseline Workshop

## Part 4: Cluster Compute

In this section, we'll examine the nodes that have been deployed in your AKS cluster.

## Concepts

Nodes are the virtual machines deloyed as part of your AKS cluster and provide the compute for your workloads. These are regular Azure Virtual Machines, deployed by default as Virtual Machine Scale Sets. Virtual machines of the same specification are deployed as a node pool. To support the deployment of different virtual machine specifications in the same AKS cluster, you can create multiple node pools. This allows you to support workloads that have different compute or storage demands, such as access to GPU's or high performance SSD's, without having to build and maintain multiple clusters.

When you create an AKS cluster the first node pool is created for you. This is known as the *system* node pool. As well as your application workloads, an AKS cluster also needs to run certain pods that are critical to the operation of the system. This includes things like a DNS service, metrics service and the tunnel service that connects to the AKS control plane. The operation of these pods should not be interfered with as it could be detrimental to the entire cluster and all of the workloads running on it, so it's considered a good practice to deploy additional node pools configured as *user* node pools.

In a multiple node pool configuration, you can set things up so that only critical system services run in the system node pool, whilst all of your application workloads run in a user node pool.

You also need separate node pools when running Windows workloads in AKS. The critical pods mentioned previously must run on Linux virtual machines, therefore the system node pool is always running Linux. To run Windows workloads, you create a new node pool that consists of Windows virtual machines.

A user node pool can be scaled down to zero nodes, but a system node pool always needs at least one node to be running.

---

## Exercises

Let's start by getting some information about the node pools in our cluster
```
az aks nodepool list --cluster-name <cluster name> -g <cluster resource group> -o table
```
The output of this command should be as follows:
>```
>Name      OsType    VmSize           Count    MaxPods    ProvisioningState    Mode
>--------  --------  ---------------  -------  ---------  -------------------  ------
>npsystem  Linux     Standard_DS2_v2  3        30         Succeeded            System
>npuser01  Linux     Standard_DS3_v2  3        30         Succeeded            User
>```

From this you can see that
- We have two node pools
- Both node pools run Linux
- One nodepool contains 3 Standard_DS2_v2 virtual machine nodes and is designated as a System pool. We use smaller virtual machines here to save on costs.
- One nodepool contains 3 Standard_DS3_v2 virtual machine nodes and is designated as a User pool. This is where our application workloads are deployed and these virtual machines will need to be sized appropriately to meet the demands of those workloads.
- We're using 3 nodes in each pool. Using at least 2 nodes will ensure that pods can be run in a highly available way with at least 2 replicas. Using 3 nodes also allows for a node to be unavailable, either for a planned event such as an upgrade of the node, or an unplanned outage.

Let's have a look at the nodes in those node pools.

```
kubectl get nodes
```

The result should be something similar to this:

>```
>NAME                               STATUS   ROLES   AGE     VERSION
>aks-npsystem-42141386-vmss000000   Ready    agent   5d15h   v1.19.6
>aks-npsystem-42141386-vmss000001   Ready    agent   5d15h   v1.19.6
>aks-npsystem-42141386-vmss000002   Ready    agent   5d15h   v1.19.6
>aks-npuser01-42141386-vmss000000   Ready    agent   5d15h   v1.19.6
>aks-npuser01-42141386-vmss000001   Ready    agent   5d15h   v1.19.6
>aks-npuser01-42141386-vmss000002   Ready    agent   4d8h    v1.19.6
>```

So, this shows six nodes in total and you'll notice that their name contains the name of the node pool each virtual machine is in.

Next, let's look at the workloads that are running on our cluster and which nodes they are running on

```
kubectl get pods -o wide -A
```
There will be a lot of information in the output from this command as we're listing all of the workloads running in all of the namespaces. Below is a truncated sample with some columns removed for ease of readability:

>```
>NAMESPACE                   NAME                                         NODE
>a0008                       aspnetapp-deployment-7d5f54499c-44j4n        aks-npuser01-42141386-vmss000000
>a0008                       aspnetapp-deployment-7d5f54499c-ms2r8        aks-npuser01-42141386-vmss000001
>a0008                       traefik-ingress-controller-89d76cbfc-swv2g   aks-npuser01-42141386-vmss000000
>a0008                       traefik-ingress-controller-89d76cbfc-tdk9g   aks-npuser01-42141386-vmss000002
>cluster-baseline-settings   aad-pod-identity-mic-cd9c49498-8dzvr         aks-npuser01-42141386-vmss000000
>cluster-baseline-settings   aad-pod-identity-mic-cd9c49498-swhx2         aks-npuser01-42141386-vmss000000
>cluster-baseline-settings   aad-pod-identity-nmi-hcjm8                   aks-npuser01-42141386-vmss000000
>cluster-baseline-settings   aad-pod-identity-nmi-n4jss                   aks-npuser01-42141386-vmss000001
>cluster-baseline-settings   aad-pod-identity-nmi-qw68w                   aks-npuser01-42141386-vmss000002
>kube-system                 coredns-b94d8b788-4d9xg                      aks-npuser01-42141386-vmss000000
>kube-system                 coredns-b94d8b788-lwwfh                      aks-npsystem-42141386-vmss000002
>```
You will notice that pods appear to be generally deployed across all of the nodes in the cluster, including the critical system components that run under the `kube-system` namespace. If we wanted to force all of the critical components to run only in the system node pool, we could do this through the use of *taints* and *tolerations*. When creating a node pool, you can specify a taint, which is like a type of label that gets applied to all the nodes in that node pool. Once a node is tainted, no workloads will be deployed there. However, when you deploy your workload, you can specify a toleration, which means that those particular workloads *can* be deployed to those nodes. The effect of this is that any workload that doesn't specify a toleration will end up being deployed to your user node pool, and critical workloads can be deployed to the system node pool.

If you examine one of the critical system components in the `kube-system` namespace, you'll see that it does indeed contain a toleration.

```
kubectl describe pod -n kube-system coredns-b94d8b788-4d9xg
```

*The name of your coredns pod will be different to that shown above, so remember to modify the command accordingly.*

Right at the bottom of the output of that command, you should see something like this

>```
>Tolerations:    CriticalAddonsOnly op=Exists
>```

All of the critical components within an AKS cluster contain this toleration, so if you apply a matching taint to the nodes in your cluster's system node pool, that will prevent application workloads being deployed there. Similarly, you can configure other node pools with taints and labels to ensure that only specified pods target specific types of node.

---

## Summary

You can use node pools to separate out different kinds of workloads in an AKS cluster and to make different specifications of virtual machines available.

---

## References

**Use multiple node pools**  
[https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools](https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools)

**Manage system node pools in Azure Kubernetes Service**  
[https://docs.microsoft.com/en-us/azure/aks/use-system-pools](https://docs.microsoft.com/en-us/azure/aks/use-system-pools)
