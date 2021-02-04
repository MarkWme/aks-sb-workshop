# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will be deploying the remaining resources that are part of the AKS Baseline. This entails the AKS Cluster and all supporting services.

---

## Part 2: Deploy the AKS cluster and supporting Azure resources

## Concepts

Instead of spending time creating all the resources line by line, we will use the output script from the ARM network deployment.

---

## Exercises

The first step is to execute the output script from section 1.

```bash
# Script output will look something like:
./1-cluster-stamp.sh westeurope aksbl02-aks-rg aksbl02-spoke-rg 11111111-1111-1111-1111-111111111111 11111111-1111-1111-1111-111111111111 /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/aksbl02-spoke-rg/providers/Microsoft.Network/virtualNetworks/vnet-spoke-BU0001A0008-00 11111111-1111-1111-1111-111111111111 11111111-1111-1111-1111-111111111111 contoso@microsoft.com contosoadmin
```

After the script has finished executing, be sure to grab the Public IP of the Application Gateway WAF to test that everything is working.

```bash
# End of Sample Output
NEXT STEPS
---- -----

1) Map the Azure Application Gateway public ip address to the application domain names. To do that, please open your hosts file (C:\windows\system32\drivers\etc\hosts or /etc/hosts) and add the following record in local host file:
    51.138.66.249 bicycle.contoso.com

2) In your browser navigate the site anyway (A warning will be present)
 https://bicycle.contoso.com

# Clean up resources. Execute:

deleteResourceGroups.sh
```

Test the Public IP endpoint using curl.

```bash
# Test Public WAF Endpoint
curl -k -H "Host: bicycle.contoso.com" https://<PUBLIC_IP_GOES_HERE>/
```

**Challenge**

- What AKS supporting resources did the deployment create?
- What is the name of the resource group that the AKS worker nodes got deployed too?

**NOTE**

- Now that you have verified provisioning completed successfully, you have a working AKS Baseline architecture. You can deploy an App to it as part of a PoC, and it can also be used for Production purposes.

---

## Summary

After successful execution of the script the AKS Baseline is setup in your Azure Subscription. Let's start exploring the baseline to see what it provides.

---

## References

N/A

---
