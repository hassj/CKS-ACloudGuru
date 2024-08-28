# Chapter 7 Monitoring, Logging, and Runtime Security

## Chapter 7.2: Understanding behavioral analytics

[falco project](https://falco.org/docs/)

### We're under attack

### Behavioral analytics
It is the process of observing the abnormal and potential malicious events what is going on you system.
Maybe use tools or manually of behavioral analytics

### Falco tools
- Falco is open source project allows you to monitor linux at runtime and alert on any suspicious activity based upon configuration rules.

- Some of use case: privileged escalate, file access, executable binari file

### Exam tips

## Chapter 7.3: Analyzing container behavior with falco

[falco output formating](https://falco.org/docs/outputs/formatting/)

[falco project](https://falco.org/docs/)

[sysdig](https://docs.sysdig.com/en/)

### falco from the command line

There are different ways to run Falco, you can run as background job, or setup web server or command line ...

`falco -r rules.yml -M 45 `

> -r: rule file
> -M : number of second 

### Falco rules
![Falco rule example](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/66-falco-example.JPG)

[Falco rules example](https://falco.org/docs/examples/)

[Falco rules git repo](https://github.com/falcosecurity/rules/blob/main/rules/falco_rules.yaml)

### Hands-on lab
- First, installing the falco on the nodes having container is running. it means that falco will be installed on all of worker nodes.
```
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | sudo apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | sudo tee -a
/etc/apt/sources.list.d/falcosecurity.list
sudo apt-get update
sudo apt-get install -y falco
```

- Creating test pod standing on control plane nodes

```
apiVersion: v1
kind: Pod
metadata:
name: falco-test-pod
spec:
nodeName: k8s-worker1
containers:
- name: falco-test
image: busybox:1.33.1
command: ['sh', '-c', 'while true; do cat /etc/shadow; sleep 5; done']

```

- On worker node, create falco rule `vi falco-rules.ym`

```
- rule: spawned_process_in_test_container
  desc: A process was spawned in the test container.
  condition: container.name = "falco-test" and evt.type = execve
  output: "%evt.time,%user.uid,%proc.name,%container.id,%container.name"
  priority: WARNING

```

- Run falco for 45 seconds

`sudo falco -r falco-rules.yml -M 45`

### Trips
- To see all available options condition output fields use --list 

## Chapter 7.5: Ensuring Containers are Immutable

[Container security context](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context-1)

### what is the Container immutability
Immutable (or stateless) containers do not change during theif life time. Instead being change, they are replaced with new containers.

### Immutability and security 

### Immutability best practices
1. Best practice to avoid elevated privileges.

- Privileged mode ``securityContext.privileged: true``

- Host Namespace `` hostNetwork: true ``

- Allow privileged escalation ``securityContext.allowPrivilegeEscalation: true``

- Runas User : root or user ID of 0 

2. Immutable containers do not change their code, which mean they could not be able to write to container file system.

3. Use readOnlyRootFileSystem: true, a container use without it also mean that its a container immutable

4. If a container need to write data to their file system, use volume mount to support this.

```
apiVersion: v1
kind: Pod
metadata:
  name: my-immutablity-pod

spec:
  containers:
  - name: my-immutable-container
    image: nginx
    securityContext:
      readOnlyRootFileSystem: true
    
    volumeMounts:
      - name: cache
        mountPath: /var/cache/nginx
      - name: log
        mountPath: /var/run
  
  volumes:
  - name: cache
    emptyDir: {}
  - name: log
    emptyDir: {}
```

### Hands-on lab

### Tips
- Immutability means that containers do not change during runtime by downloading and running new code.

- Containers have specification of privileged mode (ex: securityContext.privileged: true) may also be considered mutable

- Use container securityContext.readOnlyRootFileSystem to prevent container from writing to it files system 

- If an application needs to write to file, such as for caching or loggin, you can use an emptyDir volume alongside with readOnlyRootFileSystem.


## Chapter 7.7: Understanding Audit Logs

[Audit loging](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)

### What are audit logs?

```
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: none
  resources:
  - group: ""
    resources: ["pod", "services"]
  namespaces: ["test"]
	
```
- the audit logs are set of rules determine which event will be log and how detailed the logs are.

- level: how detailed the rule logs are, can be: 

> none: log nothing
>
> metadata: log only high level data
>
> request: log the metadata and request body
>
> requestResponse: all things such as: metadata and request body and response body.

### How audit logging works?

### Audit policy

### Tips

## Chapter 7.8: Setting up Audit Logging

[policy configuration refference](https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/#audit-k8s-io-v1-Policy)

### configuring audit logging 


### The audit policy configuration file 

To enable audit logging, just passing flag to kube-apiServer

- ``--audit-policy-file`` : points to the audit policy file

- ``--audit-log-path``: the location of log output file

- ``--audit-log-maxage``: the number of days to keep old log files

### Hands-on lab
- Create policy rules: `vi /etc/kubernetes/audit-policy.yml`

```
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
# Log changes to Namespaces at the RequestResponse level.
- level: RequestResponse
  resources:
  - group: ""
    resources: ["namespaces"]
# Log pod changes in the audit-test Namespace at Request level
- level: Request
  resources:
  - group: ""
    resources: ["pods"]
  namespaces: ["audit-test"]
# Log all ConfigMap and Secret changes at the Metadata level.
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]
# Catch-all - Log all requests at the metadata level.
- level: Metadata
```

- Enable audit-log-policy on kube-apiServer:

![Audit log policy](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/67-audit-log-policy.JPG)

![Audit log policy](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/67-audit-log-policy-2.JPG)

![Audit log policy](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/67-audit-log-policy-3.JPG)

> notice that whenever you edit kube-apiserver the cluster will recreate new pod for a while, so need take a moment for waiting response

### Tips
![Audit log policy Exam tips](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/68-Audit-log-policy-Tips.JPG)

