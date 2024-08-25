
#  CHAPTER 2. CLUSTER SETUP
## INTRODUCTION 

this chapter will cover some topic below:

- Network policies : prevent or restrict communication from or to pods

- CIS benchmark

- Ingress

- Attack surfaces

- Verifying binaries

## Restricting default access with netowrk policies

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

### Network policy review
Network policy are like firewall in k8s cluster, allow you to control network flow such as block or allow. 

### Default deny
the default of network policy when it blocks all communication between pods, except specific traffic what i need.

### Hands On lab
Default deny network policy

```
apiVersions: networking.k8s.io/v1
kind: NetworkPolicy
metatda:
  nanme: default-deny-all
  namespace: dev-example
spec:
  podSelector: {} 	// {} means all pods
  policyTypes:
  - Ingress
  - Egress
  
```
### Tips of Exam

## Allowing limited access with NetworkPolicies

[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

### Allowing traffics
```
apiVersion: networking.k8s.io/v1
kind: NetworkPoliy
metadata:
  name: my-np
  namespace: test-example
spec:
  podSelector:
    matchLabels:
	  app: nginx
  policyTypes:
  - Ingress:
  ingress:
  - from:
    - namespaceSelector:
	    matchLabels:
		  project: test
	ports:
	- protocol: TCP
	  port: 80
```

### spec.podSelector

### Ingress/ Egress rules

### Hands on Lab
Take notice different between definition below, the first one is sperate two rule in a array, the second one is combined single rule.
- Definition with one rule:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-np
  namespace: test-ns
spec:
  podSelector:
    matchLabels:
      app: nginx
  policytypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: test
      podSelector:
        matchLabels:
          app: client
    ports:
    - protocol: TCP
      port: 80
```

- Definition with two different rules:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-np
  namespace: test-ns
spec:
  podSelector:
    matchLabels:
      app: nginx		// this label will be applied to object
  policytypes:
  - Ingress
  ingress:				// this ingress used to determine source which match the condition
  - from:
    - namespaceSelector:
        matchLabels:
          project: test
    - podSelector:
        matchLabels:
          app: client
    ports:
    - protocol: TCP
      port: 80
```

## Running CIS benchmark with kube-bench

[CIS benchmark](https://www.cisecurity.org/benchmark/kubernetes)
[Kube-bench](https://github.com/aquasecurity/kube-bench)

### The CIS kubernetes benchmark
the center for internet Security (CIS) is non profit organization dedicated to promoting digital Security
CIS benhmark is a consensus driven community standards and best practice for security systems
The CIS kubernetes benchmark provides a guideline for security kubernetes environment.

### kube bench
is a tool that allow you to check your cluster how well it conform to the cis kubernetes benchmark. it will generate the report to you about the issue that occure on your cluster which mentioned on CIS benchmark documentation.

### Hands on lab
- Download this file on control plan and worker node then install: https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-master.yaml

`# wget -O kube-bench-control-plan https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-master.yaml`

![cis-kube-bench-tool-scan](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/13 - cis-kube-bench-tool-scan.jpg "cis-kube-bench-tool-scan")

## 2.6 Fixing Security Issues Detected by a CIS Benchmark

[Kube-bench](https://www.cisecurity.org/benchmark/kubernetes)
[Kube-bench-github](https://github.com/aquasecurity/kube-bench)

> some command helpfully: `kubectl delete job job_name`
>
> To remove failed jobs: `kubectl delete job $(kubectl get jobs | awk '$3 ~ 0' | awk '{print $1}') `
>
> To remove completed jobs: `kubectl delete job $(kubectl get jobs | awk '$3 ~ 1' | awk '{print $1}') `
> 
> reffer to link: [Automatically remove completed Kubernetes Jobs created by a CronJob?](https://stackoverflow.com/questions/41385403/how-to-automatically-remove-completed-kubernetes-jobs-created-by-a-cronjob)

- Run two files that you downloaded from gihub for creating 2 pod that will help you to see how well it conform the security of CIS benchmark. 
Then based on the remediation particular of each node for fixing the issue related each other. 

### Exam Tips
- Tip 1: The CIS benchmark output will help you to address the issues step by step 
- Tip 2: Kubeadm cluster use kube config file located at /var/lib/kubernetes/config.yaml on each node
- Tip 3: on kubeadm cluster, manifest files for control plane components (api server, etcd ...) can be found in /etc/kubernetes/manifests on the control plane server.

## Chapter 2.8: Implementing TLS with Ingress

[TLS in Ingress on K8s](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)

### Ingress review
That is an object put in midle external user and top of service that allow user can communicate securely with ingress then ingress can comunicate old way http with service

### Ingress and TLS
![Ingress and TLS](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/14-Ingress-tls.JPG "Ingress and TLS")

### Hands-on lab
- Create a certificate key via openssl, and delete that file once you already created that object.

`openssl req -nodes -new -x509 -keyout tls-ingress.key -out tls-ingress.crt -subj "/CN=ingress.test"`

- Base64 tls `base64 tls-cert-name.crt`

> (secret refference](https://kubernetes.io/docs/concepts/configuration/secret/)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  namespace: tls-test-namespace

spec:
  tls:
  - hosts:
      - ingress.test
    secretName: ingress-tls
  rules:
  - host: ingress.test
    http:
      paths:
      - path:
        pathType:
        backend:
          service:
            name: my-service
            port:
              number: 80
```

- result of tls-ingress (dont care about the error because of we dont create any default backend service

![tls ingress](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/15-ingress-tls-lab.JPG "TLS ingress")

## Chapter 2.10: Securing Node Endpoints
[Check required ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

### kinds of Attacks

### Minimizing the attack surface
Be awares what ports are using on your cluster or nodes. Use network segment and firewall to keeps them safe

### Listening ports
Scan port on your CLUSTER

`nc 127.0.0.1 6443 -v`

![Listenning ports on control plane node](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/16-Listenning-ports.JPG "Listenning ports")

> Most of ports above you dont want to reachable from the outside 

![Listenning ports on worker node](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/17-Listenning-ports-worker-node.JPG "Listenning ports")

### Tips

## Chapter 2.11: Securing GUI Elements
### Kubernetes GUI

[Kubernetes GUI dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

it's also an attack surface, so it shoule be control by RBAC correctly or use another tool like firewall or netowrk segment for managing access.

### Tips

## Chapter 2.12: Verifying Kubernetes Platform Binaries

[Installing kubectl binary on linux by curl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux)

### Overview kubernetes binary
- it an excutalbe file used to run k8s components. kubectl, kubeadm and kubelete all run on servers using binaries
- so if you install kubectl via package installation which is verified automatically, so if you install kubectl binary manually, you need to verify the checksum number before downloaded it.

### under attack

### Verify binaries
- check version of only client `kubectl version --short --client`
- download kubectl binary checksum: `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`
- verify checksum number: `echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check`

### Tips

## Chapter 2.14: Cluster setup review

![Cluster setup review](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/18-Cluster-setup-review.JPG "Cluster setup review")

1. Network policy
- Tip 1: defaul deny policy + targed policy to block all but necessary traffic in namespace
- Tip 2: Use empty podSelector {} to make sure apply to all pods in the namespace
- Tip 3: Pay attention to different between one rule with multiple selector and multiple rules.

2. CIS benchmark
- Tip 1: CIS kubernetes benchmark is a setup standard and practice of secure of kuberntest CLUSTER
- Tip 2: Kube bench - might be a tool that run automated to check how well your cluster conforms to the CIS benchmark
- Tip 3: kubeadm clusters use a kubelet config file located at /var/lib/kubernetes/config.yaml on each node
- Tip 4: In kubeadm cluster, manifest files for control plane components such as api-server, etcd... located at /etc/kubernetes/manifest on the control plane node.

3. Ingress
