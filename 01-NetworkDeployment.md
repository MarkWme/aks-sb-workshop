# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will be setting up a typical Hub & Spoke Network we see with most customers. For example, the Hub will contain a centralized Firewall service that will be used to govern egress traffic. The Spoke will contain the workload, AKS in our case.

---

## Part 1: Deploy the network infrastructure

## Concepts

Instead of spending time creating the network by hand, we are going to use an ARM Template to construct what we need.

---

## Exercises

The first thing we are going to do is define the variables needed for the script that will execute the ARM networking template.

```bash
# Variables
PREFIX="aksbl02"
LOC="westeurope"
SUB="<AZURE SUBSCRIPTION_GOES_HERE>"
AAD_TENANTID="<NEW_AAD_TENANTID_GOES_HERE>"
HUB_NAME_RG="${PREFIX}-hub-rg"
SPOKE_NAME_RG="${PREFIX}-spoke-rg"
AKS_RG="${PREFIX}-aks-rg"
```

After we have defined our variables, we are going to execute the **0-networking-stamp.sh** script in the **inner-loop-scripts/shell** directory in the AKS Baseline repo.

```bash
# Execute Script (inner-loop-scripts/shell)
./0-networking-stamp.sh -l $LOC -s $SUB -t $AAD_TENANTID -h $HUB_NAME_RG -p $SPOKE_NAME_RG -c $AKS_RG
```

After running this, you'll be prompted to authenticate twice. The first time is to create users in the new Azure AD tenant you created as part of the pre-requisites. The second time is to create the resources in your Azure subscription. The login ID should be the same both times. When you created the new Azure AD tenant, the ID you were logged in with at the time will have automatically been granted administrator access to the new tenant.

```bash
# Sample Output (SAVE for Next Section)
./1-cluster-stamp.sh westeurope aksbl02-aks-rg aksbl02-spoke-rg 11111111-1111-1111-1111-111111111111 11111111-1111-1111-1111-111111111111 /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/aksbl02-spoke-rg/providers/Microsoft.Network/virtualNetworks/vnet-spoke-BU0001A0008-00 11111111-1111-1111-1111-111111111111 11111111-1111-1111-1111-111111111111 contoso@microsoft.com contosoadmin
```

**NOTE**
Make sure to save the output of the script execution above for the next step on deploying resources.

---

## Summary

After successful execution of the script you should see 3 resource groups, one for the Hub Network, one for the Spoke Network, and one for the AKS Workload. Feel free to explore them.

---

## References

N/A

---
