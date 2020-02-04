# Tuesday 2/4

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

### Config Maps

Print resources

`describe` is like the pretty print version

	k get cm my-cm -o yaml

	k describe cm my-cm -o