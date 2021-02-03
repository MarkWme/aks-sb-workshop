# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will explore the integration between AKS and Azure Monitor for Containers to provide Monitoring, Logging and Observability capabilities.

---

## Part 13: Monitoring and Observability

## Concepts

Instead of spending time creating the network by hand, we are going to use an ARM Template to construct what we need.

---

## Exercises

The first thing we are going to do is define the variables needed for the script that will execute the ARM networking template.

```bash
# Variables
PREFIX="khaksbl"
LOC="westeurope"
SUB="<SUBSCRIPTION_GOES_HERE>"
AAD_TENANTID="<AAD_TENANTID_GOES_HERE>"
HUB_NAME_RG="${PREFIX}-hub-rg"
SPOKE_NAME_RG="${PREFIX}-spoke-rg"
AKS_RG="${PREFIX}-aks-rg"
```

After we have defined our variables, we are going to execute the **0-networking-stamp.sh** script in the **inner-loop-scripts/shell** directory in the AKS Baseline repo.

```bash
# Execute Script (inner-loop-scripts/shell)
./0-networking-stamp.sh -l $LOC -s $SUB -t $AAD_TENANTID -h $HUB_NAME_RG -p $SPOKE_NAME_RG -c $AKS_RG

# Sample Output (SAVE for Next Section)
./1-cluster-stamp.sh westeurope aksbl01-aks-rg aksbl01-spoke-rg <AAD_TENANTID_GOES_HERE> <SUBSCRIPTION_GOES_HERE> /subscriptions/<SUBSCRIPTION_GOES_HERE>/resourceGroups/aksbl01-spoke-rg/providers/Microsoft.Network/virtualNetworks/vnet-hub-spoke-BU0001A0008-00 6c04a532-82d6-4d39-aa1a-6d958f488e2e <AAD_TENANTID_GOES_HERE> <E-MAIL_ADDRESS_GOES_HERE> <ADMIN_GOES_HERE>
```

**NOTE**
make sure to save the output of the script execution above for the next step on deploying resources.

---

## Summary

After successful execution of the script you should see 3 resource groups, one for the Hub Network, one for the Spoke Network, and one for the AKS Workload. Feel free to explore them.

---

## References

N/A

---
