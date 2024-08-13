# CHAPTER 1. INTRODUCTION
## About the Exam 
These documentation site are allowed during the Exam

[Kubernetes documentation](https://Kubernetes.io/docs)
[Kubernetes Github](https://github.com/Kubernetes/)
[Kubernetes blog](https://Kubernetes.io/blog/)
[Trivy Docs](https://github.com/aquasecurity/trivy/)
[SysDig Docs](https://docs.sysdig.com/)
[Falco docs](https://falco.org/docs/)
[AppArmor Docs](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)

## Building a k8s cluster

[Install Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
[Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

1. Set hostname for all ( one master node and 2 worker nodes) then edit on hostfile, then enable module on kernel of node

![k8s module](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/01-Modules-install-k8s.jpg "module")

2. install containerd packages

`sudo apt-get update && sudo apt-get install containerd -y`

3. Setup an container configuration file.
![containerd configuration](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/02-containerd-configuration-install-k8s.jpg "containerd configuration")

4. Disable swap
...

5. install network plugin 

![containerd configuration](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/02-containerd-configuration-install-k8s.jpg "containerd configuration")

6. Adding worker node to cluster
![Generating token](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/08-Generate-token-adding-node.jpg "Generating token")
![Adding node to cluster](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/09-adding-node.jpg "Adding node to cluster")

## Kubernetes security overview
Layered security model focuses on making each layer as secure as possible.
![Layered security model](https://github.com/hassj/CKS-ACloudGuru/tree/main/Image/10-layerd-security-model.jpg "Layered security model")

1. The cluster

If someone can control the cluster they could gain the application.

2. Host Systems

The hosts system and infrastructure used to run the cluster could serve as an attacker vector

3. Application

The containered application and code could be compromised and used to interact with other application

4. Supply chain

Container image and other files used to deploy application could be used by attacker.