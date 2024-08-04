# CHAPTER 1. INTRODUCTION
## About the Exam 
These documentation site are allowed during the Exam

[Kubernetes documentation](https://Kubernetes.io/docs)
[Kubernetes Github](https://github.com/Kubernetes/)
[Kubernetes blog](https://Kubernetes.io/blog/)
[Trivy Docs](https://github.com/aquasecurity/trivy/)
[SysDig Docs](https://docs.sysdig.com/)
[Falco docs](https://falco.org/docs/)
[AppArmor Docs](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation

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

Reading: 
https://learn.acloud.guru/course/certified-kubernetes-security-specialist/learn/f9432f75-a4cf-439b-80fc-9df4ff640e18/de85fbaa-1e4e-4c35-b0e8-13c467173879/watch
