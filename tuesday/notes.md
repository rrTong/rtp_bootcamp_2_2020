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


##

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

