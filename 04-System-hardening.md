# Chapter 4: System hardening
## chapter 4.1: System hardening intro

https://learn.acloud.guru/course/certified-kubernetes-security-specialist/learn/41ed7deb-bab7-4c1f-ae92-9dff074ea65a/cefa1bfa-95f9-4130-ace3-8b1bc5def138/watch

- Host os security

- IAM role

- network security 

- apparmor

## Chapter 4.2: Understanding Host OS Security Concerns

[Security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

[security context v1](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#securitycontext-v1-core)

[pod spec v1](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#podspec-v1-core)

### we've under attack

The attacker can gain control a pod then keeping control host server, so far he coul take over all of the pod that node. 

> Protect the k8s host from container running on it.

### Host namespace
Container use OS namespace to isolate themselves from other container and the host (it different form a k8s namespace)

> You can configure Pods to use host namespace instead use container namespace isolated with host namespace (in case your container need to interact directly with host) but it will be comes with security risk

![Host namespace](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/24-host-namespace-1.JPG "Host namespace")

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  hostIPC: true
  hostNetwork: true
  hostPID: true
  containers:
  - name: first-pod
    image: nginx
```

- There are 3 parameter which by default set to false, those are: ``hostIPC - host inter process communicate``, ``hostNetwork - network's host`` , `` hostPID if set to true, container use process id of host, can communicate directly to host``

![Host namespace](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/24-host-namespace-2.JPG "Host namespace")

> Use settings like hostIPC, hostPID, hostNetwork only when absolutely necessary.

### Privilege mode
![privilege-mode](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/25-privilege-mode.JPG "privilege-mode")

```
apiVersion: v1
kind: Pod
metadata: 
  name: my-pod
spec:
  containers:
  - name: fist-pod
    image: nginx
	securityContext:
	  privileged: true
```

> privilege-mode set to false by default

![privilege-mode](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/25-privilege-mode-2.JPG "privilege-mode")

### Tips

## Chapter 4.3: Minimizing IAM Roles

[aws security best practice ](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)

[iam role on aws](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

### IAM Roles

![IAM role](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/26-IAM-role.JPG "IAM role")

### Protecting amazone resources
- kubernetes containers running on aws maybe able to access IAM credentials, so need to be aware off:
> priciple of least privilege
> 
> block access if do not need - for ec2 if dont need use IAM credential you can block access from IP: 169.254.169.254

## Chapter 4.4: Exploring Network-Level Security

[cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

### the cluster network

![cluster network](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/26-cluster-network.JPG "cluster network")

### Minimizing cluster network acccess

- Anyone who can access to cluster network can comunicate with any pod and service in the cluster

- when possible, limit access to the cluster network from outside

## Chapter 4.5: Exploring AppArmor

[Apparmor documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)

[Apparmor documentation](https://help.ubuntu.com/community/AppArmor)

> link above will guide you to disable sepcific apparmor profile or disable apparmor framework

### what is apparmor?

Linux security kernel module

![Apparmor](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/27-Apparmor.JPG "Apparmor")


### Apparmor profile

is a set of rules that defines what a program can do and cannot do

![Apparmor profile](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/28-Apparmor-profile.JPG "Apparmor profile)

An apparmor profile can be loaded to apparmor at the server level and activated in one of two modes

- Complain mode: just generating report what are going on of program

- Enforcing mode: it will actively prevent the program from doing anything that the profile does not allow.

> profile need to be enabled on the server the pod is running on, or the pod won't start.

### Tips

- it is a linux kernel security module that allows grandular control over that individual program do and can not do.

- two modes working of apparmor profile

## Chapter 4.6: Using AppArmor in k8s Containers

[restricting container's access to resource using apparmor](https://kubernetes.io/docs/tutorials/security/apparmor/)

[Apparmor documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)

### Enforcing apparmor profile
An apparmor profile can be stored in a file and applied using command ``apparmor_parser``

`sudo apparmor_parser /path/to/file`

> By default apparmor will be load to enforce mode, using -C flag to load  complain modes

### Applying apparmore profile to Container
- use annotation to apply apparmor profiles to specific containers

- The annotation name should begin with ``container.apparmor.security.beta.kubernetes.io/`` , followed by the name of the container.

- Or You can specify the appArmorProfile on either a container's securityContext or on a Pod's securityContext.

- The annotation value should be localhost/ (this case apparmor profile is gonna be enabled on localhost), followed by the name of apparmor profile
```
apiVersion: v1
kind: Pod
metadata: 
  name: my-pod-applied-Apparmor-profiles
  annotation:
    container.apparmor.security.beta.kubernetes.io/nginx: localhost/k8s-deny-write
spec:
  containers:
  - name: nginx
    image: nginx
	
```

### Hands-on lab

- Apparmor will be reload automatically after server reload.

```
# sudo vi /etc/apparmor.d/deny-write 	// put that file to this path for loading profile to reffer that file on pod manifest file
> then chown root:root for that file 

#include <tunables/global>
profile k8s-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}

# sudo apparmor_parser /etc/apparmor.d/deny-write	// load apparmor-profile server

```

- Creating simple pod without using apparmor profile
```
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-disk-write
  
spec:
  containers:
  - name: my-pod
    image: nginx
	command: ['sh', '-c', 'while true; do echo "I write to the disk" > diskwrite.log; sleep 5; done']

# kubectl create -f apparmor-dis-write-pod.yml --save-config

```

- Load apparmor-profile to specific pod

```
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-disk-write
  annotation:
    container.apparmor.security.beta.kubernetes.io/my-pod: localhost/deny-write
  
spec:
  containers:
  - name: my-pod
    image: nginx
	command: ['sh', '-c', 'while true; do echo "I write to the disk" > diskwrite.log; sleep 5; done']

# kubectl create -f apparmor-dis-write-pod.yml --save-config
```

- you can not add apparmor profile without deleting the existing pod and creating the pod.

`kubectl delete pod apparmor-disk-write --force`

then recreating apparmor-pod with annotation added.. you will get the error bellow since either worker node have not apparmor profile, to fix this, just enalbe apparmor-profile on worker nodes.

![apparmor-hands-on-lab](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/29-Apparmor-profile-hands-onlab.JPG "apparmor-hands-on-lab")

![apparmor-hands-on-lab](https://github.com/hassj/CKA-acloudguru/blob/main/CKA-md/Image/29-Apparmor-profile-hands-onlab2.JPG "apparmor-hands-on-lab")

### Tips

- Use apparmor_parser to load an apparmor profile from a file. It will be load the profile in enforcing mode by default

- Use pod annotations to apply apparmor profile to specific container. for example ``container.apparmor.security.beta.kubernetes.io/name_of_container: localhost/name_of_apparmor_profile ``

## chapter 4.8: System Hardening Review

### Host OS security 
- Tip 1: Protect your hosts from attack that might come from within containers

- Tip 2: Be aware of Pod settings like: hostIPC, hostNetwork, hostPID. Use them only when absolutely necessary.

- Tip 3: Be aware of using privileged mode for container with securityContext.privileged.

### IAM role
- Tip 1: if your container application running on kubernetes of AWS, your container application maybe able to access to IAM credentials.

- Tip 2: Avoid providing unnecessary permission to IAM roles (principle of least privilege)

- Tip 3: If your application container does not need IAM access, consider blocking access to IAM credential via firewall, network policy, etc.

### Network security 
- Tip 1: By default, anyone who can access cluster network can communicate with all Pods and service in cluster. 

- Tip 2: when possible, limit access to the cluster network form outside

### Apparmor 

- Tip 1: Apparmor is a linux kernel security module that allows you granular to controls over individual programs can and cannot do.

- Tip 2: use apparmor_parser command to load profile from a file, by default it will be enabled in enfoce mode.

- Tip 3: Use annotation to apply apparmor profile to specific pod container. for example: `container.apparmor.security.betat.kubernetes.io/nginx: localhost/deny-write `
