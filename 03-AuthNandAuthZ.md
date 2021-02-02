# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment
In parts 1 and 2 of this workshop you completed the deployment of an AKS cluster and various supporting Azure services. In this and the remaining parts of the workshop, we will dive deeper into the various components to help you understand how they work and why they are used.

---

## Part 3: Authentication and Authorisation

---

## Concepts

Kubernetes clusters support two types of users. One of these is service accounts, which are managed by the Kubernetes API and generally used by pods to allow in-cluster processes to talk to the Kubernetes API. 

The other account type is normal users. Kubernetes does not have objects which represent normal user accounts and they cannot be added to the cluster through an API call. Instead, it's expected that a cluster-independent service manages these users.

By default, authentication is provided through certificates. Any user presenting a valid certificate signed by the cluster's certificate authority is considered to be authenticated. However, certificate management can be a chore, especially if you're going to start managing certificates for everyone who needs to access your cluster. Most things in Kubernetes are pluggable and the authentication process is no exception. This means you can plug in custom authentication methods.

The Kubernetes API server has an OIDC (Open ID Connect) plug-in and Azure AD supports OIDC. Therefore, you can configure your Kubernetes cluster to work with Azure AD, then users and groups can be granted access to the cluster and the lifecycle of those users and groups is managed in Azure AD as usual. 

Access to a Kubernetes cluster's resources is managed through it's own Role Based Access Control (RBAC) process. This is separate from Azure AD's RBAC (although there is a preview feature that maps Azure AD RBAC roles to Kubernetes RBAC). When you deployed the cluster, you provided the object ID for an Azure AD group that you'd previously created. As part of the process of enabling an AKS cluster for use with Azure AD, the deployment will have created Kubernetes RBAC objects and configured them to allow that Azure AD group to have administrator level access to the cluster. 

You can add more users to that Azure AD group to grant them administrator access to the cluster, or you can create new Azure AD groups and then use Kubernetes RBAC to allow those groups specific access rights to certain resources. Using this method, you could, for example, ensure that developers and cluster operators only have the necessary access level to the Kubernetes resources related to their own applications and not to those belonging to other applications.

---

## Exercises

Your cluster has had Azure Active Directory integration enabled. The easiest way to see what this means, is to try and do something on your cluster.

First, let's find the details of your newly deployed cluster. Type the following command

```
az aks list -o table
```

You will see a list of AKS clusters. Find the cluster you just deployed and note its name and resource group name.

![az aks list -o table command and output](images/03azakslist.png)

We will now get the access credentials needed for our cluster.

```
az aks get-credentials -n <your cluster name> -g <your cluster's resource group>
```

After a few seconds, you should see a message that tells you your cluster has been merged as current context

```
Merged "aks-oqchal7ga453i" as current context in /home/mark/.kube/config
```
The `kubectl` command gets its configuration from what's known as the *kubeconfig* file. This is normally a file named `config` stored in the hidden `.kube` folder under a user's home directory. In the output from `az aks get-credentials` you should see the full path to your kubeconfig file - i.e `/home/mark/.kube/config` in this example. You can store details for multiple clusters in kubeconfig. The *current context* simply means that the cluster specified is now the default and all `kubectl` command you run will target that cluster.

If you examine your kubeconfig file, you'll find a section similar to the following:

```
users:
- name: clusterUser_akssb-cluster_aks-oqchal7ga453i
  user:
    auth-provider:
      config:
        apiserver-id: 6dae42f8-4368-4678-94ff-3960e28e3630
        client-id: 80faf920-1908-4b52-b5ef-a8e7bedfc67a
        config-mode: '1'
        environment: AzurePublicCloud
        tenant-id: 72f988bf-86f1-41af-91ab-2d7cd011db47
      name: azure
```
This is how Azure AD authentication is configured. The `apiserver-id` value will match an Enterprise Application object that will be visible in your Azure AD tenant. This object will be named *Azure Kubernetes Service AAD Server*. The `client-id` refers to an internal AAD object that you wont see in your tenant! The `tenant-id` will be the Tenant ID for your Azure AD instance.

Now let's see what happens when we try to run a command against our cluster. Try the following

``` kubectl get pods -A ```

This should list all of the pods running in your cluster across all namespaces. However, what you should see is a prompt similar to the following

> To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AH8HHMZFX to authenticate.

This is because your cluster has AAD integration enabled and you need to authenticate using an AAD account. Go ahead and open [](https://microsoft.com/devicelogin) and enter the code. Remember your code will be different to the one documented here!  You'll then be asked to authenticate with Azure AD.

- Use the Azure AD account that you added to the AKS management group

![Azure Active Directory sign in prompt](images/03aadsignin.png)

- 

kubectl describe clusterrolebinding aks-cluster-admin-binding-aad

---

## References

**Overview of Kubernetes Authentication (Kubernetes Documentation)**  
[https://kubernetes.io/docs/reference/access-authn-authz/authentication/](https://kubernetes.io/docs/reference/access-authn-authz/authentication)

**Open ID Connect**  
[https://openid.net/connect/](https://openid.net/connect/)

**Open ID Connect authentication with Azure Active Directory**  
[https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/auth-oidc](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/auth-oidc)

**Microsoft Identity Platform and OpenID Connect Protocol**  
[https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-protocols-oidc](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-protocols-oidc)

**Use Azure RBAC for Kubernetes Authorisation**  
[https://docs.microsoft.com/en-us/azure/aks/manage-azure-rbac](https://docs.microsoft.com/en-us/azure/aks/manage-azure-rbac)