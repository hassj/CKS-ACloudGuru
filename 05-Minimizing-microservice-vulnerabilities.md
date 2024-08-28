# CHAPTER 5: Minimizing Microservice Vulnerabilities

## Chapter 5.1: Minizing microsevice Vulnerabilties

![Microservice Vulnerabilities](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/30-Microservice-vulnerabilities.JPG "Microservice Vulnerabilities")

## Chapter 5.2 Managing Container Access with Security Contexts

[container Security context](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context-1)

[Pod security context](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context)

[configuring security context for container or pod](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### What is a security context
A portion of container or pod specification that allows you to provide set of special security or access control setting at the pod an container level.

- spec.securityContext - configures the security context at the Pod level, these settings apply to all containers in the pod.

- runAsUser: set OS custom ID for container Process 

- spec.containers[].securityContext - sets securityContext at the container level.

### Configuring security context

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-security-context
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: my-pod-security-context
    image: nginx
	securityContext:
	  runAsUser: 2000
	  
```

### Hands-on lab

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-security-context
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: my-pod-security-context
    image: nginx
	securityContext:
	  allowPrivilegeEscalation: false 
	  
```
### Tips

- tip 1: securityContext offer variety of security settings or access control releated setting.

- Tip 2: spec.securityContext sets Security for all the container under pod definition

- Tip 3: spec.containers[].securityContext sets security setting for specific container, this will be applied to individual containers within the Pod


## Chapter 5.3 Governing Pod configuration with pod security policies

[Pod security polices- deprecated](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/)

[Pod security admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#podsecuritypolicy)

[Pod security policies](https://kubernetes.io/docs/concepts/security/pod-security-policy/)

### We're under attack
Sometimes one of devs deploy application into k8s cluster without implement application Security, so you need to enforce the application Security configuration automatically, you need pod Security policies.

### What is a pod security policies
- It allows k8s cluster administrator to control what Security-related configuration pod are allowed to run with.

- Use them to automatically enforce desired Security configuration within the cluster.

### How does pod security policy work 
![Pod security policies example](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/31-Pod-security-policies-example.JPG "Pod Security policies example")

> Pod Security polices can change the pod by providing defaults for certain values.

- Pod Security polices can control things like:
1. pivilege mode : Allow or disallow user running ocntainer in privileged mode 
2. host namespace: block hostnamespace setting like ``hostNetowrk= true``
3. runAsUser/runAsGroup : run as specific user/group or just prevent running as root.
4. Volumes: control with volume types which are allowed for pod storage volume.
5. allowedHostPaths: limit hostPath volumes to only specific path

![Pod security policies can do](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/32-pod-security-policies-can-do.JPG "Pod security policies can do")

> Now pod Security polices are being deprecated and will be replaced by different k8s functionality in the future.

### Tips

## Chapter 5.4 Using Pod Security Policies
[pod security rules](https://kubernetes.io/docs/concepts/security/pod-security-policy/)

[Admission controllers refference](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#podsecuritypolicy)

### The PodSecurityPolicy Admission controller
To use Pod Security polices you must first activate pod security policy admission controller. 

- Activate the podsecuritypolicy admission controller by adding --enable-admission-plugins flag for kubernetes Api server.

- A pod must satisfy at least one pod security police in order to be allowed. If you activate podsecuritypolicy admission controller before creating any policies, you maynot create any pod or it will be not allowed.

### Creating Pod security policies
PodSecurityPolicy is a k8s object, it's also use defintion yaml format.

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-psp
spec:
  privileged: false
  runAsUser:
    rule: RunAsAny
	
```

> privileged - determines whether Pods created using policy can run container in privileged mode
>
> runAsUser: Determines which user the container in the pod can run as. So runAsAny allow container run as any user 

### Authorizing policies

![Two-ways-authorize-policies](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/33-Authorized-policy.JPG "Two-ways-authorize-policies")

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-cr-use-psp
rules:
- apiGroups: ['policy']
  resource: ['podsecuritypolicy']
  verbs: ['use']
  resourceName:
  - my-psp
```

- For a user to use podsecuritypolicy to validate their pod, they must be authorized to use that policy via rbac
- the use Verb in a role ro clusterRole allow a user to use podsecuritypolicy
- A new pod created must be allowed by at least one podsecuritypolicy which the user is authorized to use. If user is not authorized to use any policy, they cannot to create any pod.

### Hands-on lab

![Pod Security policy Hands-on lab](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/35-pod-security-policy-hands-on-lab.JPG "Pod security policy hands-on lab")

- First, on the control plane node we enable security policy admission controller in file ``/etc/kubernetes/manifest/kube-apiserver.yml``

![Admission controller](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/34-admission-controller.JPG "Admission controller")

- Create podsecuritypolicy non privileged

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: my-psp-nonpriv
spec:
  privileged: false
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  selinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - persistentVolumeClaim
  - secret
  - projected

```

- Creating clusterRole allow ServiceAccount to use above policy

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-use-psp-nonpriv
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolies']
  verbs: ['use']
  resourceNames:
  - my-psp-nonpriv
  
```

- Creating roleBinding to bind that clusterRole to ServiceAccount

```
apiVersion: rbac.authorization.k8s.io/v1
kind: roleBinding
metadata:
  name: rb-use-psp-nonpriv
  namespace: psp-test
roleRef:
  kind: ClusterRole
  name: cr-use-psp-nonpriv
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name:
  namespace: psp-test

```

- Then create a pod without any special setting, you will successfully with that action. Since you are using a user with admin permission. the user with admin permission will be able to create pod successfully automatically.

- this time create Pod using service account

![Pod use service account](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/36-Creating-Pod-using-ServiceAccount.JPG "Pod use service account")

- And now create simple pod using service Account but with an violation permission bellow: 

![Pod use service account violation](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/37-Creating-Pod-using-ServiceAccount-violation.JPG "Pod use service account violation")

## Chapter 5.5: Using OPA Gatekeeper

[Gatekeeper documentation](https://github.com/open-policy-agent/gatekeeper "Gatekeeper documentation")

### What is OPA Gatekeeper
- Open policy Agent (OPA) Gatekeeper allows you to highly-customizable policies on any kind of k8s objects at creation time.

- Policies are defined using on OPA constraint framework

- so essentially, OPA is a framework define constraint and requirement arount objects, and OPA Gatekeeper allows us to use that tool to define policies for objects creation in kubernetes

- Example

![OPA Gatekeeper example](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/38-OPA-gatekeeper-example.JPG "OPA gatekeeper example")

> so, you could define any policies or requirement for any objects in creation time using OPA Gatekeeper.

![OPA gatekeeper deny creating deployment example](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/38-OPA-gatekeeper-example-2.JPG "OPA gatekeeper deny creating deployment example")

### Constraint template

![Constraint Template](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/39-Constraint-template.JPG "constraint template")

```
apiVersion: templates.gatekeeper.sh./v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
        listKind: K8sRequiredLabelsList
        plural: k8srequiredlabels
        singular: k8srequiredlabels
      validation:
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: string 
  targets: 
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlables

        violation[{"msg": msg, "detail": {missing_labels}}] {
        provided := {label | input.review.object.metadata.labels[label]}
        required := {label | label := input.parameters.labels[_]}
        missing := required - provided 
        count(missing) > 0
        msg := sprintt("you must provide labels: %v", [missing])
        }
```
- Defines the schema for a constraint and the Rego will enforce the Constraint.

### Constraint

![Constraint example](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/40-Constraint-example.JPG "constraint example")

```

apiVersion: constraint.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: dep-must-have-contact
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Deployment"]
  parameters:
    labels: ["contact"]
```

## Chapter 5.6: Managing Kubernetes Secrets

[Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

### What is secret?
store and manage the sensitive data such as passwork, api token, and certificates, this data can be passed to containers at runtime.

- type: k8s supports multiple types that specify the way store data. ``Opaque`` is the default type, and it allows to store arbitary user-defined data.
- data: this block contain the values are being encode as base64: it also are key-value map data

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: fdatweFGAG
  password: cGGDafdgaga

```
### Seret data in environment variable 
![Secret as environment](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/42-secret-environment.JPG "secret as environment")

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    env:
    - name: my-user-secret
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: my-password-secret
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password

```

### Secret data in volume
![secret data as volume](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/43-secret-data-in-volume.JPG "secret data as volume")

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-busy-pod
    image: busybox
    volumeMounts:
    - name: my-secret-vol
      mountPath: "/etc/credentials"
      readOnly: true

volumes:
  - name: my-secret-vol
    secret:
      secretName: my-secret
```
### Retrieving secret data 
![Retrieve secret data](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/44-Retrieve-secret-data.JPG "Retrieve secret data")

```
kubectl get secret <secret_name> -o yaml 		// to get base64 encode secret data
echo <base64_data> | base64 --decode  			// to decode encode secret data.
```

### Hands-on 
- encoded raw data 

![secret in pod](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/45-secret-example.JPG "secret in pod")

- paste in the pod definition

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  restartPolicy: Never
  containers:
  - name: pod-using-secret-var
    image: busybox
    command: ['sh', '-c', 'echo "the credential are: $USERNAME:$PASSWORD"']
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-scret
          key: password
```
### Tips

- secrets store sensitive data such as: password, username, api token...

- you can pass secret data to running container at both ways, either environment variable or mounted as volume 

- you can retrieved secret data from command line, you can use ``kubectl get secret -o yaml`` to get the encoded base64 data, then decode it with base64 --decode command.

## Chapter 5.8: Understanding Container Runtime Sandboxes

[Run time Class](https://kubernetes.io/docs/concepts/containers/runtime-class/)

[Kata container Documentation](https://katacontainers.io/docs/)

[gVisor Documentation](https://gvisor.dev/docs/)

### what is container runtime sanbox ?
![Container runtime sanbox](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/46-Container-runtime-sanbox.JPG "Container runtime sanbox")

It is a special container runtime that providing extra layer of process isolation and greater security.

![Container runtime sanbox Use cases](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/47-Container-runtime-sanbox-use-casese.JPG "Container runtime sanbox use cases")

Run container runtime sanbox in situations where you need extra security around containers such as: untrust workload, multi tenant, small and simple 

> The extra security of container runtime sanbox usually comes at the cost of performance, so that why we dont implement container runtime sanbox eveywhere

### gVisor/runsc
![gvisor/runsc](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/48-gvisor-runSC.JPG "gvisor/runsc")

- gvisor is a linux kernel application that simulate kernel within host kernel.

- runsc is an OCI-runtime container that integrates gvios with application like k8s.

### Kata containers
![kata container](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/49-kata-container.JPG "kata container")

### Tips

## Chapter 5.9: Creating a Container Runtime Sandbox

### Building a sanbox 
1. Install gVisor runtime

2. configure containerd

- need to configure containerd to interact with runsc

3. Create a runtime Class

you will need to create runtime class which will allow you to designate which pods need to use the sandbox runtime.

### Runtime Class

it will the pod what runtime to use.

```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: my-runtimeclass
handler: my-configuration
```

- handler: refers to section in containerd configuration file that configure runsc

Use runtime class in pod 

```
apiversion: v1
kind: Pod
metadata:
  name: my-pod

spec:
  runtimeClassName: my-runtimeclass
  containers:
  - name: my-container
    image: nginx

```

- use runtime class in pod under spec of pod with ``runtimeClassName`` to provide runtime class to pod and allow it to use sanboxed runtime.

### Hands-on lab

- install gviros on all nodes (add GPG key onto your system, add repository to your OS)

```
curl -fsSL https://gvisor.dev/archive.key | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64,arm64] https://storage.googleapis.com/gvisor/releases release
main"
sudo apt-get update && sudo apt-get install -y runsc
```

- edit containerd configuration and add configuration runsc

`sudo vi /etc/containerd/config.toml`

- Find the disabled_plugins section and add the restart plugin

`disabled_plugins = ["io.containerd.internal.v1.restart"]`

- Find the block [plugins."io.containerd.grpc.v1.cri".containerd.runtimes] . After the existing runc block, add
configuration for a runsc runtime. It should look like this when done:
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
runtime_type = "io.containerd.runc.v1"
runtime_engine = ""
runtime_root = ""
privileged_without_host_devices = false
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
runtime_type = "io.containerd.runsc.v1"
```

- Locate the [plugins."io.containerd.runtime.v1.linux"] block and set shim_debug to true .

```
[plugins."io.containerd.runtime.v1.linux"]
...
shim_debug = true		//debug be enaabled will help you to troubleshooting when something go wrong.
```

- Restart containerd, and verify that it still runs (to get the containerd to pickup the changes, you need to restart by systemctl)

```
sudo systemctl restart containerd
sudo systemctl status containerd
```

- From this point, the remainder lab will only be completed only on control plane node. Create runtime class 

```
# vi  runsc-sandbox.yml

apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: my-rcs
  
handler: runsc
```

- then create two pods with sandbox and no sanbox environment

```
apiVersion: node.k8s.io/v1
kind: Pod
metadata:
  name: pod-sanbox
spec:
  runtimeClassName: my-rcs
  containers:
  - name: my-pod
    image: busybox
	command: ['sh', '-c', 'while true; do echo "running ..." ; sleep 5; done ']
```

> dmesg command basically print out kernel message.

![runtime sandbox hands-on lab 1](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/52-runtime-sanbox-hands-on-lab-1.JPG "runtime sandbox hands-on lab 1")

![runtime sandbox hands-on lab 2](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/52-runtime-sanbox-hands-on-lab-2.JPG "runtime sandbox hands-on lab 2")

![runtime sandbox hands-on lab error](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/52-runtime-sanbox-hands-on-lab-error.JPG "runtime sandbox hands-on lab error")


### Tips

- Use runtime sanbox to create a specialized container runtinme configuration, such as one that will use gVisor/runsc

- Set the runtimeClassName property under pod specification to make pod use the container runtime sanbox.

## Chapter 5.11: Understanding Pod-to-Pod mTLS

[Mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication)

[Manage TLS certs in a cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

### We're under attack

![mtls is](https://github.com/hassj/CKS-ACloudGuru/blob/main/Image/53-mtls.JPG "mtls is")

> https is secure method that only one way, in which server have sertificate and client just trust that certification without private key, 
> 
> mtls is two ways authenticate with each other and ecrypted each communication.

### Certificates signing 
- k8s api allows you to obtain certificates which you can use in your app.

- Certificate authority be provided by API will be generated certificate for client, you can trust that CA.

- You can obtain certificates programmatically using k8s API.

### Tips
- You can obtain certificates programmatically using K8S API.

## Chapter 5.12: Signing Certificates

[Managing TLS certificate in a cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

[Creating CSR by openssl tool](https://phoenixnap.com/kb/generate-openssl-certificate-signing-request)

### process of requesting and signing certificates

- The requestor creates a Certificate Signing Request object to request a new Certificate.

- The CSR can be approved or denied.

- Permissions related to certificate signing can be managed by rbac

### demonstration

54-Signing-certificate-hands-on-lab.JPG

54-Signing-certificate-hands-on-lab-2.JPG

- Creating the csr object in k8s 

54-Signing-certificate-hands-on-lab-3.JPG

54-Signing-certificate-hands-on-lab-4.JPG

54-Signing-certificate-hands-on-lab-5.JPG

54-Signing-certificate-hands-on-lab-6.JPG

### Tips

## Chapter 5.13: Minimizing Microservice Vulnerabilities Review

### Security context
offer a variety of security setting and access controll setting 

### Pod security Policies

### OPA Gatekeeper

### secrets

### Runtime sandbox

### mtls and Certificates

