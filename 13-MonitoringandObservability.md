# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment

In this part we will explore the integration between AKS and Azure Monitor for Containers to provide Monitoring, Logging and Observability capabilities.

---

## Part 13: Monitoring and Observability

## Concepts

The Azure Monitor for Containers (AzMon for Containers) feature is the recommended tool for monitoring and logging because you can view events in real time. It captures container logs from the running pods and aggregates them for viewing. It also collects information from Metrics API about memory and CPU utilization to monitor the health of running resources and workloads. You can use it to monitor performance as the pods scale. Another advantage is that you can easily use Azure portal to configure charts and dashboards.

It has the capability to create alerts that trigger Automation Runbooks, Azure Functions, and others.

Most workloads hosted in pods emit Prometheus metrics. Azure Monitor is capable of scraping Prometheus metrics and visualizing them.

---

## Exercises

Let's explore the AzMon for Containers solution and what you get out of the box.

First, let's take a look at how the logs get from the cluster to Log Analytics.

```bash
# How do the logs get there?
kubectl -n kube-system get po -l component=oms-agent

# What does the omsagent use to send logs to Log Analytics?
kubectl -n kube-system get configmap omsagent-rs-config -o yaml
```

Second, let's take a look at what kind of data is captured. For that let's visit the Azure Portal:

1. Open Azure Portal
2. Go to AKS Cluster
3. Click on Insights Blade & Explore
4. Click on Logs Blade & Explore

Try the following KQL in the Logs Blade, what is it doing?
AzureDiagnostics
| where Category == 'kube-audit-admin'
| extend auditevent=parse_json(log_s)
| extend objectRef = auditevent.objectRef
| distinct tostring(objectRef.resource)

**Challenge**

- Instead of using "kubectl logs ..." to search logs, find another way.
- Query the **cluster-autoscaler** Control Plane logs. **Hint: They are captured using Azure Diagnostics.**

**NOTE**

- Feel free to explore more and try adding your own Flux agent setup to the cluster.

---

## Summary

Congratultions! You made it to the end. You now have a complete understanding of all the different aspects of the AKS Baseline Architecture.

---

## References

- [Azure Monitor for Containers](https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-overview)
- [AKS Control Plane Logs](https://docs.microsoft.com/azure/aks/view-control-plane-logs)
- [AKS Baseline Monitoring](https://docs.microsoft.com/azure/architecture/reference-architectures/containers/aks/secure-baseline-aks#monitor-and-collect-metrics)

---
