# Tuesday 2/4

[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/configuration/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/configuration/)

## Summary of Commands

kubectl to k
	
	alias k=kubectl

List pods

	k get pods

Apply pod yaml

	k apply -f foo.yaml

Delete pod

	k delete my-pod

List config maps

	k get cm

Apply config map yaml

	k apply -f cm.yaml

List logs

	k logs my-pod

## Notes

Sometimes pods won't show unless you add `-A` tag

	k get pods

	k get pods -A

List all namespaces
	
	k get ns

Create namespace named `bar`
	
	k create ns bar

Clean pod list

	k delete pod myapp-pod

List pods

	k get pods

Namespace `bar` does not appear unless `-n bar` added	

	k get pods -n bar
	
#### Trick for yaml

Put this between two sections to create separate yaml files

	---
`foo.yaml`

	apiVersion: v1
	kind: Namespace
	metadata:
		name: bar
	---
	apiVersion: v1
	kind: Pod
	metadata:
		name: myapp-pod
		namespace: bar
		labels:
			app: myapp

	
	k apply -f foo.yaml

This is okay too

	k apply -f https://...

## Config Maps

Print resources

`describe` is like the pretty print version

	k get cm my-cm -o yaml

	k describe cm my-cm -o

Get logs

	k logs my-pod

Edit config map
Change value, then save in editor then it'll update automatically

	k edit cm my-cm

Validate that the config map has been changed

	k describe cm my-cm

Can delete and create a new pod to update it

	k delete pod my-pod

### Using foo3.yaml

Apply yaml to pod 	
	
	k apply -f foo.yaml

Observe pod

	k describe pod

Observe Environment Variables

	k logs my-pad
* ^ note that 

		color=black
		location=naboo

Restart pod

If container seems to be running, `k exec`

Two ways to pass information
* environment variables
* volume

## 

`k completion -h` teaches you how to get autocomplete on kubectl

`restartPolicy` applies to all containers

Find out what containers do

	k explain Pod.spec.containers

Output:

	KIND:     Pod
	VERSION:  v1

	RESOURCE: containers <[]Object>

	DESCRIPTION:
	     List of containers belonging to the pod. Containers cannot currently be
	     added or removed. There must be at least one container in a Pod. Cannot be
	     updated.
     ...

### using foo4.yaml

No need for config map

	k apply -f foo.yaml
	k logs my-env-pod

Output:

	Hello from the environment

### using foo5.yaml

	k apply-f foo.yaml
	k edit pod my-inter-pod 
	k logs my-inter-pod

(didn't edit file during `k edit`)
output:

	minikube my-inter-pod 172.17.0.5

To get more information, can add `-o wide`

	k get pod my-inter-pod
	k get pod my-inter-pod -o wide

output:

	NAME           READY   STATUS      RESTARTS   AGE
	my-inter-pod   0/1     Completed   0          2m56s

vs

	NAME           READY   STATUS      RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
	my-inter-pod   0/1     Completed   0          2m29s   172.17.0.5   minikube   <none>           <none>

Delete all pods in current namespace

	k delete pods -all

Be sure not to delete important things such as service

List services

	k get service

#### Resource limits
`foo.yaml` `spec.containers:`

	limits:
		memory: "128Mi"
		cpu: "500m"

If not enough memory, container gets killed and will auto restart; may often 400 / 500
If not enough CPU, response will get throttled and take slower

Adding worker nodes and removing working nodes cost money, but it's easy in Kubernetes
Ideally have a system that automatically scales up and down