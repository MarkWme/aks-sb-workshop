# Azure Kubernetes Service Secure Baseline Workshop

## Examining the AKS Secure Baseline Deployment
In parts 1 and 2 of this workshop you completed the deployment of an AKS cluster and various supporting Azure services. In this and the remaining parts of the workshop, we will dive deeper into the various components to help you understand how they work and why they are used.

---

## Part 3: Authentication and Authorisation

## Concepts

Kubernetes clusters support two types of users. One of these is service accounts, which are managed by the Kubernetes API and generally used by pods to allow in-cluster processes to talk to the Kubernetes API. 

The other account type is normal users. Kubernetes does not have objects which represent normal user accounts and they cannot be added to the cluster through an API call. Instead, it's expected that a cluster-independent service manages these users.

By default, authentication is provided through certificates. Any user presenting a valid certificate signed by the cluster's certificate authority is considered to be authenticated. However, certificate management can be a chore, especially if you're going to start managing certificates for everyone who needs to access your cluster. Most things in Kubernetes are pluggable and the authentication process is no exception. This means you can plug in custom authentication methods.

The Kubernetes API server has an OIDC (Open ID Connect) plug-in and Azure AD supports OIDC. Therefore, you can configure your Kubernetes cluster to work with Azure AD, then users and groups can be granted access to the cluster and the lifecycle of those users and groups is managed in Azure AD as usual. 

Access to a Kubernetes cluster's resources is managed through it's own Role Based Access Control (RBAC) process. This is separate from Azure AD's RBAC (although there is a preview feature that helps map Azure AD RBAC roles to Kubernetes RBAC). When you deployed the cluster, you provided the object ID for an Azure AD group that you'd previously created. As part of the process of enabling an AKS cluster for use with Azure AD, the deployment will have created Kubernetes RBAC objects and configured them to allow that Azure AD group to have administrator level access to the cluster. 

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

Next, we get the access credentials needed for our cluster.

```
az aks get-credentials -n <your cluster name> -g <your cluster's resource group>
```

After a few seconds, you should see a message that tells you your cluster has been merged as current context

>```
>Merged "aks-oqchal7ga453i" as current context in /home/mark/.kube/config
>```
The `kubectl` command gets its configuration from what's known as the *kubeconfig* file. This is normally a file named `config` stored in the hidden `.kube` folder under a user's home directory. In the output from `az aks get-credentials` you should see the full path to your kubeconfig file - i.e `/home/mark/.kube/config` in this example. You can store details for multiple clusters in kubeconfig. The *current context* simply means that the cluster specified is now the default and all `kubectl` commands you run will target that cluster.

If you examine your kubeconfig file, you'll find a section similar to the following:

>```
>users:
>- name: clusterUser_akssb-cluster_aks-oqchal7ga453i
>  user:
>    auth-provider:
>      config:
>        apiserver-id: 6dae42f8-4368-4678-94ff-3960e28e3630
>        client-id: 80faf920-1908-4b52-b5ef-a8e7bedfc67a
>        config-mode: '1'
>        environment: AzurePublicCloud
>        tenant-id: 72f988bf-86f1-41af-91ab-2d7cd011db47
>      name: azure
>```
This is how Azure AD authentication is configured. The `apiserver-id` value will match an Enterprise Application object that will be visible in your Azure AD tenant. This object will be named *Azure Kubernetes Service AAD Server*. The `client-id` refers to an internal AAD object that you wont see in your tenant! The `tenant-id` will be the Tenant ID for your Azure AD instance.

Now let's see what happens when we try to run a command against our cluster. Try the following

```
kubectl get pods -A
```

This should list all of the pods running in your cluster across all namespaces. But what you will see is a prompt similar to the following
>```
>To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AH8HHMZFX to authenticate.
>```
Because your cluster has AAD integration enabled, you now need to authenticate using an AAD account. Go ahead and open [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) in your browser and enter the code. Remember your code will be different to the one documented here!  You'll then be asked to authenticate with Azure AD.

> Remember to authenticate using the Azure AD account that you added to the AAD group that you created.

![Azure Active Directory sign in prompt](images/03aadsignin.png)

Once authentication is complete, return to the terminal where you entered the `kubectl` command and you should see the output.

>```
>NAMESPACE                   NAME                                         READY   STATUS    RESTARTS   AGE
>a0008                       aspnetapp-deployment-7d5f54499c-44j4n        1/1     Running   0          4d8h
>a0008                       aspnetapp-deployment-7d5f54499c-ms2r8        1/1     Running   0          4d8h
>a0008                       traefik-ingress-controller-89d76cbfc-swv2g   1/1     Running   0          4d8h
>a0008                       traefik-ingress-controller-89d76cbfc-tdk9g   1/1     Running   0          3d10h
>cluster-baseline-settings   aad-pod-identity-mic-cd9c49498-8dzvr         1/1     Running   0          3d10h
>... (and several more lines like this!)
>```
As well as Azure AD authentication, Kubernetes RBAC was used for authorisation. Kubernetes has *ClusterRoles* (roles that are applied across the whole cluster) and *Roles* (roles that are applied to a specific namespace). We then have ClusterRoleBindings and RoleBindings that are used to map users or groups to roles.

> **Remember**  
> Authentication = Who are you?  
> Authorisation = What can you do?  

When your AKS cluster was deployed, Kubernetes RBAC would have been automatically configured to allow the Azure AD group you created to have adminstrator level permissions on the cluster. Let's take a look at that.

First, we'll find the ClusterRoleBinding resources that the AKS deployment created
```
kubectl get clusterrolebindings | grep "aks"
```
The output should be as follows:
>```
>aks-cluster-admin-binding              ClusterRole/cluster-admin               4d22h
>aks-cluster-admin-binding-aad          ClusterRole/cluster-admin               4d22h
>aks-service-rolebinding                ClusterRole/aks-service                 4d22h
>system:aks-client-node-proxier         ClusterRole/system:node-proxier         4d22h
>system:aks-client-nodes                ClusterRole/system:node                 4d22h
>```
Next, we'll examine the `aks-cluster-admin-binding-aad` ClusterRoleBinding in more detail
```
kubectl describe clusterrolebinding aks-cluster-admin-binding-aad
```
This will provide the details for this particular binding as follows:
>```
>Name:         aks-cluster-admin-binding-aad
>Labels:       addonmanager.kubernetes.io/mode=Reconcile
>              kubernetes.io/cluster-service=true
>Annotations:  <none>
>Role:
>  Kind:  ClusterRole
>  Name:  cluster-admin
>Subjects:
>  Kind   Name                                  Namespace
>  ----   ----                                  ---------
>  Group  56e5d9db-602e-49cb-a594-e7bd3c00a7f3
>```
As you can see, the `ClusterRole` named `cluster-admin` is bound to a `Group` whose `Name` should be the object ID of the AAD group you created. This is effectively saying that we want to grant this "Group" the role of "cluster-admin"

We can also examine cluster-role to see what level of access that grants us
```
kubectl describe clusterrole cluster-admin
```
The result will look like this
>```
>Name:         cluster-admin
>Labels:       kubernetes.io/bootstrapping=rbac-defaults
>Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
>PolicyRule:
>  Resources  Non-Resource URLs  Resource Names  Verbs
>  ---------  -----------------  --------------  -----
>  *.*        []                 []              [*]
>             [*]                []              [*]
>```
Effectively, this role is granting full access to all resources on the cluster.

Kubernetes RBAC rules can be very granular. As an experiment, have a look at the pre-defined `view` ClusterRole

```
kubectl describe clusterrole view
```
The output from this command is quite long. Here is some of the output
>```
>Name:         view
>Labels:       kubernetes.io/bootstrapping=rbac-defaults
>              rbac.authorization.k8s.io/aggregate-to-edit=true
>Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
>PolicyRule:
>  Resources                                    Non-Resource URLs  Resource Names  Verbs
>  ---------                                    -----------------  --------------  -----
>  <-- snip -->
>  pods/log                                     []                 []              [get list watch]
>  pods/status                                  []                 []              [get list watch]
>  pods                                         []                 []              [get list watch]
>  <-- snip -->
>```
Here we're being very specific about the types of resources that we want to permit access to and the verbs (actions) that we can take against those resources. So, for example, a user with the `view` role can only get, list or watch a pod. Effectively, this gives them read-only access.

---

## Summary

The advantages of using Azure AD to manage user access to an AKS cluster should be clear, especially when compared to the default option Kubernetes provides of managing access via certificates. Using Azure AD allows for effortless alignment with corporate identity management policies, simplifying the joiner, mover, leaver process and the lifecycle of a user's identity. Combined with Kubernetes RBAC, you can configured fine grained roles and bindings to ensure your users only have access to the resources they need and can control exactly what they are able to do with them.

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