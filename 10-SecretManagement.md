# Azure Kubernetes Service Secure Baseline Workshop

## Part 10: Secret Management

In this section, we cover the use of secrets and how they are managed in the secure baseline.

## Concepts

In other sections, we've covered how Azure Key Vault is used to securely store secrets. Whilst Kubernetes has it's own secrets management mechanism built in, using Azure Key Vault offers a more robust solution which can align with secret management policies used within your organisation.

In the secure baseline, Key Vault is configured with private link and uses a private endpoint on the subnet where the AKS cluster nodes are located. This limits network access to Key Vault and only allows the cluster nodes access from an internal network.

Access to secrets in Key Vault is secured using Azure AD identities. In this case, two managed identities are being used to retrieve certificates from Key Vault. One for Azure Application Gateway and one for the Traefik ingress controller.

Traefik is running in pods on the AKS cluster and makes use of an additional component called the Azure Key Vault provider for Secrets Store CSI Driver.

CSI (Container Storage Interface) is a standard for exposing arbitrary block and file storage systems to containerized workloads on Kubernetes. Using CSI third-party storage providers can write and deploy plugins exposing new storage systems in Kubernetes without ever having to touch the core Kubernetes code.

The Secrets Store CSI driver allows Kubernetes to mount multiple secrets, keys, and certs stored in enterprise-grade external secrets stores into their pods as a volume. Once the Volume is attached, the data in it is mounted into the container's file system.

---

## Exercises

Let's examine how Traefik is able to access a Key Vault secret to obtain the certificate used for decrypting traffic.

In the Azure portal, find the Key Vault instance and select the Access Policies blade.

![Key Vault Access Policies](/images/10-kvaccess.png)

The `podmi-ingress-controller` identity has been granted `Get` access for secrets and certificates.

Azure AD Pod Identity creates custom resources (CRD's) of type `AzureIdentity` in Kubernetes that represent managed identities. You can view the custom resource like this

```
kubectl describe azureidentity -n a0008 podmi-ingress-controller-identity
```
The output will look like the following
>```
>Name:         podmi-ingress-controller-identity
>Namespace:    a0008
>Labels:       <none>
>Annotations:  <none>
>API Version:  aadpodidentity.k8s.io/v1
>Kind:         AzureIdentity
>Metadata:
>  Creation Timestamp:  2021-01-29T08:56:57Z
>  Generation:          1
>  Managed Fields:
>    API Version:  aadpodidentity.k8s.io/v1
>    Fields Type:  FieldsV1
>    fieldsV1:
>      f:metadata:
>        f:annotations:
>          .:
>          f:kubectl.kubernetes.io/last-applied-configuration:
>      f:spec:
>        .:
>        f:clientID:
>        f:resourceID:
>        f:type:
>    Manager:         kubectl-client-side-apply
>    Operation:       Update
>    Time:            2021-01-29T08:56:57Z
>  Resource Version:  109127
>  Self Link:         /apis/aadpodidentity.k8s.io/v1/namespaces/a0008/azureidentities/podmi-ingress-controller-identity
>  UID:               e2228623-ba57-4c18-a82d-ebbb4b8c6197
>Spec:
>  Client ID:    fd5092e4-dbdd-4148-af75-2d294076a6d2
>  Resource ID:  /subscriptions/808121c2-95b0-4e15-8dbc-cb6de76a956a/resourceGroups/akssb-cluster/providers/Microsoft.ManagedIdentity/userAssignedIdentities/podmi-ingress-controller
>  Type:         0
>Events:         <none>
>```

This is defining an AzureIdentity named `podmi-ingress-controller-identity` and it references the `podmi-ingress-controller` managed identity in Azure that we looked at earlier.

Next, a binding is setup to map the AzureIdentity into a pod. You can view this as follows:

```
kubectl describe AzureIdentityBinding -n a0008 podmi-ingress-controller-binding
```

This will produce output similar to this
>```
>Name:         podmi-ingress-controller-binding
>Namespace:    a0008
>Labels:       <none>
>Annotations:  <none>
>API Version:  aadpodidentity.k8s.io/v1
>Kind:         AzureIdentityBinding
>Metadata:
>  Creation Timestamp:  2021-01-29T08:56:57Z
>  Generation:          1
>  Managed Fields:
>    API Version:  aadpodidentity.k8s.io/v1
>    Fields Type:  FieldsV1
>    fieldsV1:
>      f:metadata:
>        f:annotations:
>          .:
>          f:kubectl.kubernetes.io/last-applied-configuration:
>      f:spec:
>        .:
>        f:azureIdentity:
>        f:selector:
>    Manager:         kubectl-client-side-apply
>    Operation:       Update
>    Time:            2021-01-29T08:56:57Z
>  Resource Version:  109128
>  Self Link:         /apis/aadpodidentity.k8s.io/v1/namespaces/a0008/azureidentitybindings/podmi-ingress-controller-binding
>  UID:               a830aa5f-9831-4461-b6d9-c931ca128a11
>Spec:
>  Azure Identity:  podmi-ingress-controller-identity
>  Selector:        podmi-ingress-controller
>Events:            <none>
>```

The important part is at the bottom. It's effectively saying that any pod with the `Selector` or label that matches `podmi-ingress-controller` will be bound to this identity.

We want to use the identity with the Traefik ingress controller. If you run the `kubectl describe pod` command and specify the namespace `a0008` and the name of one of your Traefik ingress pods, you will see in the output that it mentions in the `Labels` section `aadpodidbinding=podmi-ingress-controller`. This is how the Azure AD Pod Identity service is injecting the Azure Managed Identity into the Traefik pod.

Now, let's see how the CSI secrets driver actually gets the secret from Key Vault and gets it to the Traefik pod. The CSI secrets driver creates another custom resource, this time called a `SecretProviderClass`. This is then configured with details of how to retrieve the secret we're interested in. You can view this using

```
kubectl describe secretproviderclass -n a0008 aks-ingress-contoso-com-tls-secret-csi-akv
```

The interesting part of the output is at the end
>```
>Spec:
>  Parameters:
>    Keyvault Name:  kv-aks-oqchal7ga453i
>    Objects:        array:
>  - |
>    objectName: traefik-ingress-internal-aks-ingress-contoso-com-tls
>    objectAlias: tls.crt
>    objectType: cert
>  - |
>    objectName: traefik-ingress-internal-aks-ingress-contoso-com-tls
>    objectAlias: tls.key
>    objectType: secret
>
>    Tenant Id:         72f988bf-86f1-41af-91ab-2d7cd011db47
>    Use Pod Identity:  true
>```

The `Keyvault Name` references the Azure Key Vault we're going to access to retrieve the secret. Then in the `Objects` section we define the specific objects we're interested in retrieving and how they're going to be represented in the pod - in this case as files named `tls.crt` and `tls.key`.

In the definition of the Traefik pods, we specify a storage volume mount that relates to the CSI secrets driver. This is the final part of the process and will result in the secrets being made available to the pod as files. If we again use the `kubectl describe pod` command and specify the namespace `a0008` and the name of one of the Traefik pods, we can see a couple of interesting things in the output.

In the `Mounts `section, we see

>```
>    Mounts:
>      /certs from ssl-csi (ro)
>```

And in the `Volumes` section we see

>```
>  ssl-csi:
>    Type:              CSI (a Container Storage Interface (CSI) volume source)
>    Driver:            secrets-store.csi.k8s.io
>    FSType:
>    ReadOnly:          true
>    VolumeAttributes:      secretProviderClass=aks-ingress-contoso-com-tls-secret-csi-akv
>```

So, the effect of this is that the secret defined in the `SecretProviderClass` object is going to end up being mounted in the pod's `/certs` directory as two files named `tls.crt` and `tls.key`.

Let's take a look at the Traefik pods to see this. We'll use an `exec` command to run a shell inside one of the Traefik pods

```bash
kubectl exec -n a0008 <name of one of your Traefik pods> -t -i -- /bin/sh
```

After a few moments you should see a prompt. Change into the certs directory and list its contents.

```bash
/ $ cd certs
/certs $ ls -al
total 12
drwxrwxrwt    2 root     root            80 Feb  5 08:21 .
drwxr-xr-x    1 root     root          4096 Feb  5 08:21 ..
-rw-r--r--    1 root     root          1237 Feb  5 08:21 tls.crt
-rw-r--r--    1 root     root          2941 Feb  5 08:21 tls.key
```
You can see the two files containing the certifcate and private key are present and ready for Traefik to use.

---

## Summary

Azure Key Vault can be used to provide a secure place to store secrets. We can use private endpoints to control network access and Azure AD identities to specify who can access the vault. Through managed identities, Azure AD Pod Identity and the Secrets Store CSI driver, we can securely access secrets and use them in appications running in a cluster.

---

## References

**Azure Key Vault Provider for Secrets Store CSI Driver**  
[https://github.com/Azure/secrets-store-csi-driver-provider-azure](https://github.com/Azure/secrets-store-csi-driver-provider-azure)

**Secrets Store CSI Driver**  
[https://github.com/kubernetes-sigs/secrets-store-csi-driver](https://github.com/kubernetes-sigs/secrets-store-csi-driver)
