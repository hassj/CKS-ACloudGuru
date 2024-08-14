# Chapter 3: CLUSTER HARDENING
## Chapter 3.1: Cluster Hardening Intro

## Chapter 3.2 Service account 

[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

[Configure Service account for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

[Managing Service Account](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)

![Role and RBAC](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/19-RBAC-Roles.JPG "Role and Rbac")

![Service Account](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/20-Service-Account.JPG "Service Account")

![Under attack](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/21-Uder-attack.JPG "Under attack")

### RBAC
If an attacker get control a pod, then he could keep controlling all of cluster via service account with high permission, so we need to limit and restrict permission of service account via RBAC

> a rolebinding can be used to bind a clusterrole in context of a specific namespace

## Restricting Service Account permission

[Using RBAC Authorization] (https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

[Configure Service Account for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

[Managing Service Account](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)

> Make sure service account have only the necessary RBAC permission

### Hands-on lab

## Chapter 3.5: Restricting access to kubernetes API

[controlling access to api](https://kubernetes.io/docs/concepts/security/controlling-access/)

kubernetes API is an http interface to the CLUSTER

> the attacker can use user account compromise or exploit the k8s api directly via port 6443
>
> so, just limiting the user access via stand point and limiting access to api via segment network and firewall that's all necessary way to do for securing k8s cluster.

## Chapter 3.6: kubernetes update
![k8s update](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/22-k8s-update.JPG "k8s udpate")

[udpate skew version](https://kubernetes.io/releases/version-skew-policy/)

## Chapter 3.7: Cluster hardening review

![Cluster hardening review](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/23-cluster-hardening-review.JPG "Cluster hardening review")

