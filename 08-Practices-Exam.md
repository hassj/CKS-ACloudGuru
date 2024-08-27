# Chapter 8 Practice Exam

## CIS benchmark

some of issues mentioned in CIS benchmark documentation:

########################################################

 kube-apiserver
 
 Turn off profiling by adding the --profiling=false flag to the kube-apiserver

 Change the authorization mode from AlwaysAllow to Node, RBAC

########################################################

 kubelet

 Below authentication, change the anonymous value from enabled: true to enabled: false to disable anonymous authentication

 Ensure Webhook mode is enabled for authentication and authorization, setting enabled to true and authorization mode to Webhook

########################################################

 etcd

 Enable client-cert-auth by setting the flag value to true
 
## Restricting default access with network policies

Create a network policy called 'mtdoom-np' that allows specific traffic on port 80 to reach the 'mtdoom' Pod in the mordor namespace.

The policy should allow incoming traffic:

From all Pods in the 'frodo' namespace. From all Pods with the label 'app=sam', regardless of which namespace they are in.


```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mtdoom-np
  namespace: mordor
spec:
  podSelector:
    matchLabels:
      app: mtdoom
  policyTypes: ["Ingress"]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          app: frodo
    - podSelector:
        matchLabels:
          app: sam
      namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 80
```

![Practice lab](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/69-practice-lab.JPG)
