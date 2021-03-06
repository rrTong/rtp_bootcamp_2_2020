# Tuesday 2/4

[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/configuration/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/configuration/)

## Summary of Commands

kubectl to k
	
	alias k=kubectl

Command | Kind
------- | ----- 
apply     | pod
get   | configmap (cm)
delete  | namespace (ns)
describe| service
edit    | limitrange
logs | secrets
exec | serviceaccount
top | node

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

	k describe cm my-cm -o yaml

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

## Resource Requirements

### Using foo6.yaml

`foo.yaml` `spec.containers:`

	limits:
		memory: "128Mi"
		cpu: "500m"

If not enough memory, container gets killed `oom` and will auto restart; may often 400 / 500
If not enough CPU, response will get throttled and become slower

Adding worker nodes and removing working nodes cost money, but it's easy in Kubernetes
Ideally have a system that automatically scales up and down

If you don't specify `resources.requests` or `resources.limits`:

### Using resources.yaml

	k apply -f resources.yaml
	k get limitrange

Kubernetes never scales pod, always scales container

## Secrets

	# Create files needed for rest of example. 
	echo -n 'admin' > ./username.txt 
	echo -n '1f2d1e2e67df' > ./password.txt
	cat username.txt
	> admin
	cat password.txt
	> 1f2d1e2e67df
	kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt

kubectl
	
	k get secrets
	k get secrets db-user-pass
	k get secrets db-user-pass -o yaml
	k get secrets db-user-pass -o json
	echo YWRtaW4= | base64 -D
	> admin

### Using foo7.yaml

Change `data.username` from  `YWRtaW4=`:
	
	data:
		username: <base64_replaceme>

So that you don't store plaintext PII in repository
Using base64 is a must

### Using foo8.yaml

	k apply -f foo.yaml
	k get secrets mysecret-config -o yaml
	> apiVersion: v1
	  data:
	    config.yaml: YXBpVXJsOiAiaHR0cHM6Ly9teS5hcGkuY29tL2FwaS92MSIKdXNlcm5hbWU6IHRva2VuCnBhc3N3b3JkOiB0aGVzZWNyZXR0b2tlbg==
	echo YXBpVXJsOiAiaHR0cHM6Ly9teS5hcGkuY29tL2FwaS92MSIKdXNlcm5hbWU6IHRva2VuCnBhc3N3b3JkOiB0aGVzZWNyZXR0b2tlbg== | base64 -d
	> apiUrl: "https://my.api.com/api/v1"
	  username: token
	  password: thesecrettoken

To chain `get` arguments

	k get secrets,cm

Sometimes you need to delete the pod to remove override complications

### Using foo9.yaml

	k get pods
	k delete pod my-pod
	k apply -f foo.yaml

`k exec` to run shell inside container

	k exec -it my-pod bash
	> ...
	  SECRET=USERNAME=admin
	  ...
	> ...
	  APIUrl: "https://my.api.com/api/v1"
	  username: token
	  password: thesecrettoken

## Security Contexts

### Using foo10.yaml

	securityContext:
		runAsUser: 2001
		runAsGroup: 3001
		fsGroup: 3001

## Service Accounts
	kubectl create sa my-service-account
	k get sa
	k describe serviceaccount my-service-account
	
### Using foo12.yaml

	k apply -f foo.yaml
	k describe pod my-pod
	k get pod -o yaml
	k get pod my-pod -o yaml
	> ...
	  serviceAccountName: myServiceAccount

## Lab

[Pod Creation](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab1/)

[Pod Config](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab2/)

## Sidecar Design Pattern

Sidecar Implementation shares the same IP Address (localhost)

	READY
	2/2

To retrieve info from another container 

	k get pods
	k exec -it my-pod bash
	curl http://localhost:8080
	> Hello from the side container

Sidecar implementation is good for circuit breaking, service mesh, certificates, etc.
However if done incorrectly, it adds extra load and it doesn't scale well 

## Liveness and Readiness Probes

[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/observability/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/observability/)

The probe constantly checks the status to see the status of the container

* ExecAction
	* simple command to check if container is alive
* TCPSocketAction 
	* just check if this socket is open
	* i.e. check that port 8080 is open
* HTTP
	* declare endpoint, if returns 200 then it's okay
	* i.e. /live /help /healthz

### Using liveness.yaml
	
	k apply -f foo.yaml
	k get pods
	k describe pod my-pod -o yaml

Example in yaml

	livenessProbe:
		tcpSocket:
			port:  8080
		initialDelaySeconds:  10
	readinessProbe:
		httpGet:
			path: /
			port:  8080
		periodSeconds:  10

## Container Logging

### Using container.yaml

Usually not just 1 container running, so aggregate logs across containers

Gives all the logs of every container

	stern .

Good to see all logs in a separate windows to debug

To check all the logs with "counter" in the name.

	stern counter .
	
## Monitoring Applications

	minikube addons enable metrics-server
	get pods -A -w
	> NAMESPACE     NAME                               READY   STATUS              RESTARTS   AGE
	  default       my-pod                             1/1     Running             1          80m
	  kube-system   coredns-5644d7b6d9-k2zpv           1/1     Running             0          132m
	  kube-system   coredns-5644d7b6d9-nq2pr           1/1     Running             0          132m
	  kube-system   etcd-minikube                      1/1     Running             0          131m
	  kube-system   kube-addon-manager-minikube        1/1     Running             0          132m
	  kube-system   kube-apiserver-minikube            1/1     Running             0          132m
	  kube-system   kube-controller-manager-minikube   1/1     Running             0          131m
	  kube-system   kube-proxy-s87nk                   1/1     Running             0          132m
	  kube-system   kube-scheduler-minikube            1/1     Running             0          131m
	  kube-system   metrics-server-6754dbc9df-2svt9    0/1     ContainerCreating   0          5s
	  kube-system   storage-provisioner                1/1     Running             0          132m
	  web           website-pod                        1/1     Running             0          123m
	  kube-system   metrics-server-6754dbc9df-2svt9    1/1     Running             0          6s

Not immediate, takes time for metrics to be collected

### Using monitor1.yaml, monitor2.yaml
	k apply -f monitor1.yaml
	k apply -f monitor2.yaml
	k describe node
	k top pods
	k top nodes

## Lab

[Probes](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab4/)