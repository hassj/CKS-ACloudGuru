# CKS note

## Network policy

1. Use a default deny network policy to block all traffic to and/or from Pods in a namespace
2. Use "{}"  podSelector to make the policy apply to all Pods in the namespace
3. Use the policyType field to block incomming or outgoing traffic or both

## Allowing limited access with Network policy
1. If a policy allows a particular traffic, it will be allowed. It means that you can add targeted policy alongside with the default deny policy to allow necessary traffic.
2. Use a policy's podselector to targeted the policy to specific pods based upon the labels 


