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

![Practice lab](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/69-practice-lab.JPG)


## Using Trivy to scan Image

` kubectl get pod -n sunnydale -o custom-columns="NAME:.metadata.name, IMAGE:.spec.containers[*].image" `

`trivy image -s HIGH,CRITICAL <image_name>`

## Apparmor profile

take a note that, need to load apparmor on each nodes which you need to use it.


> Take a notice about the apparmor and falco both need to load/enable/installed on each node that need to be scaned.

## Audit log, and monitoring cluster

- Monitoring cluster with falco. Falco need to be installed on each nodes which pod running on.

- Falco rules
```
- rules: name_of_rule
  desc: describe rule
  condittion: container.name = "test" and evt.type = execve
  output: "%evt.time, %user.id, %proc.name, %container.id"
  priority: WARNNING
```

- Audit log: need to be enabled on kube-apiServer with 3 parameters:
``--audit-log-path``, ``audit-policy-file``, ``audit-log-maxage``, ``audit-log-maxbackup``

## working with secret

```
apiVersion: v1
kind: Pod
metadata:
  name: my-Pod
  namespace: larry
spec:
  containers:
  - name: busybox
    image: busybox
	command: ["sh","-c","...]
	volumeMounts:
	- name: credential
	  mountPath: /etc/credentials
	  readOnly: true
  volumes:
  - name: credential
    secret:
	  secret_name: secrete_name_fiel
	
```
## Immutable container

Take note of the configuration setting for ``allowPrivilegedEscalation`` and ``runAsUser``

## runtimeClass and gVisor/runsc or kata container application

![gvisor-hands-on-lab](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/gvisor-hands-on-lab.JPG "gvisor-hands-on-lab")


# Usefull refference link:

[kubernetes command commonly](https://technekey.com/customizing-the-kubectl-output/)


> Take a notice about the apparmor and falco and gVisor all they need to load/enable/installed on each node that need to be scaned.