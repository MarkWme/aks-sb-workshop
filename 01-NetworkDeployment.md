# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will be setting up a typical Hub & Spoke Network we see with most customers. For example, the Hub will contain a centralized Firewall service that will be used to govern egress traffic. The Spoke will contain the workload, AKS in our case.

---

## Part 1: Deploy the network infrastructure

## Concepts

Instead of spending time creating the network by hand, we are going to use an ARM Template to construct what we need.

---

## Exercises

The first thing we are going to do is define the variables needed for the script that will execute the Network ARM Template.
./0-networking-stamp.sh -c khaksbl-rg -h khhub-rg -l westeurope -s 812f1ba2-73fd-40c2-8ee5-f8f59d4a6e7d -t 72f988bf-86f1-41af-91ab-2d7cd011db47 -p khspoke-rg
```bash
# Variables
PREFIX="khaksbl"
LOC="westeurope"
SUB="<SUBSCRIPTION_GOES_HERE>"
SUB="812f1ba2-73fd-40c2-8ee5-f8f59d4a6e7d"
AAD_TENANTID="<AAD_TENANTID_GOES_HERE>"
AAD_TENANTID="72f988bf-86f1-41af-91ab-2d7cd011db47"
HUB_NAME_RG="${PREFIX}-hub-rg"
SPOKE_NAME_RG="${PREFIX}-hub-rg"
AKS_RG="${PREFIX}-aks-rg"
```

After we have defined our variables, we are going to execute the **0-networking-stamp.sh** script in the **inner-loop-scripts/shell** directory in the AKS Baseline repo.

```bash
./0-networking-stamp.sh -l $LOC -s $SUB -t $AAD_TENANTID -h $HUB_NAME_RG -p $SPOKE_NAME_RG -c $AKS_RG
```

---

## Summary

After successful execution of the script you should see 3 resource groups, one for the Hub Network, one for the Spoke Network, and one for the AKS Workload.

---

## References

N/A

---
