# Monday 2/3

[https://ibm.box.com/s/dgz6x8nqfd61d7p59y520g1bhmgoc7zo](https://ibm.box.com/s/dgz6x8nqfd61d7p59y520g1bhmgoc7zo)

## Summary of Commands


## Notes

Find container named “myapp”

	docker ps -a | grep myapp

Build docker image:
	
	docker build -f Dockerfile -t app .

Expose ports 3000, 8000, 8443

	docker run --name mynginx -p 3000:3000 8000:8000 8443:8443 mynginx_image

expose it to `localhost:8080`

	docker run --name mynginx -p 8080:8080 app

tag to push to Dockerhub / IBM Cloud

	docker tag bitnami/nginx:1.16 [docker.io/csantanapr/mynginx:1.16](http://docker.io/csantanapr/mynginx:1.16)

	docker login -u kubeadmin -p $(oc whoami -i) $IMAGE_REGISTRY

## More Docker commands

login

	docker login -u csantanapr -p hispassword

push

	docker push docker.io/csantanapr/mynginx:1.16

pull

	docker pull docker.io/csantanapr/mynginx:1.16

run from Dockerhub

	docker run -p 8080:8080 docker.io/csantanapr/mynginx:1.16

## Environment Variables

	docker run -e FOOKEY=BAR1 -e FOOKEY2=BAR2 fookey
^ fookey being the name of the image

Execute (exec), interactive (-it) which runs bash inside the container

	docker exec -it $(container id) bash

Is

	docker exec $(container id) ls

env

	docker exec $(container id) env

env and search for “FOO” string
	
	docker exec $(container id) env | grep FOO

## 

	docker run -v $PWD/data:/data mynginx_image

Gives all properties of container in JSON

	docker inspect $(container id)

## IBM Cloud Hello World exercise

	docker pull us.icr.io/zohiba/hello:latest
	docker run us.icr.io/zohiba/hello

## Kubernetes
	
	minikube config view  
	minikube set memory 4096  
	minikube set kubernetes-version v1.16.6  
	minikube config set cpus 4
		
	minikube status
	> host: Running  
	  kubelet: Running  
	  apiserver: Running  
	  kubeconfig: Configured

## Avoid typos!

	alias k=kubectl

## yaml fun

Apply `yaml` file, minikube takes yaml and converts to JSON..  
if yaml is changed, must be re`apply`ed`-v` verbose  
`-v6` shows logs  
`-v8` shows more logs

	k apply -f pod.yaml
	k apply -f pod.yaml -v8

kubernetes 

	k get pods

To see yaml  

	k describe pod $(podname)

Open in editor  

	k edit pod myapp-pod

Change above editor to open in VS Code  

	export KUBE=EDITOR='code -w'

## kubectl apply vs kubectl create
apply - makes incremental changes  
create - overwrites all changes
([More Info](https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create))

## 

`--restart=Never` gives a pod rather than a deployment  
`--dry-run` won’t do it unless `-o yaml`  

k run nginx --image=nginx --restart=Never --dry-run  
k run nginx --image=nginx --restart=Never --dry-run -o yaml

Get all events in this namespace
	
	k get events

`k describe` and `k get` are swiss army knife  
two commands to use for many things w/ pods

## Namespaces

Display all namespaces  

	k get ns

Create new namespace  

	k create ns kabanero

	k get ns foo -o yaml

^foo being the name of a namespace

	k config set-context —current --namespace=foo