# Wednesday 2/5

## Talk w/ Wendy && Kirk

first cpat engagement:
$700k SMBC bank, includes 4 cloudpaks

Vegas:
learn how to network
get mentors, senseis, senior leads
bigger picture of ibm, sense of ibm garage, cloudpak familiarity
how to get cpaters learning about java and websphere, apiconnect, p-series

skills will be mapped with CPAT as a whole
5 = world class

need 80% of room to be able to install redhat, cloudpak, on prem

[Carlos' Awesome list of learning resources.](https://github.com/csantanapr/awesome-learn)

if we ready to install ibm cloud for MCM, and can talk, we will be able to deploy

sellers and tech sellers will contact the management team
as work succeeds, the work will come to us

learn **ibm websphere** + java (springboot), websphere liberty
cloud nativization
app modernization - existing code, 

smaller companies usually want to make big changes fast, the top 10% of engagements
big companies usually move slower, they want to change but they have risks to take

## Lab

[Manage Multiple Containers](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab3/)

# Notes
[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/pod-design/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/pod-design/)
[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/services-networking/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/services-networking/)
## Labels, Selectors, and Annotations

### Using foo2.yaml

`labels` in kubernetes are used to define things, group things

	labels:
		app: foo
		tier: frontend
		env: dev

`annotations` are used by tools

	annotations:
		imageregistry: "https://hub.docker.com/"
		gitrepo: "https://github.com/csantanapr/knative"

annotate to put metadata when an object doesn't have the schema

	k apply -f foo.yaml
	k get pods
	k get pods --show-labels

`--show-labels` will display labels within `get pods`

	LABELS
	app=foo,env=dev,tier=frontend

Using labels `-l` will help you find certain pods

	k get pods -l app -A
	k get pods -l tier=frontend
	> NAME     READY   STATUS    RESTARTS   AGE
	  my-pod   1/1     Running   0          50s

Make sure you get the labels right, otherwise you may have debugging/troubleshooting fun time with incorrect labeling

## Deployments

Often Carlos deletes pods because of pod name conflicts

	k delete pods --all

### Using foo3.yaml

	labels:
		app: nginx

Indicates to always have 3 replicas

	spec:
		replicas: 3

`matchLabels` is a query to match specific deployment to pod
wants the deployment (namespace resource) to have a pod with the matching label `app: nginx`

	selector:
		matchLabels:
			app: nginx

---
#### Q&A
Carlos doesn't memorize kubectl commands

	kubectl | grep resources
	> ...
	  api-resources
	kubectl | grep api-

For instance, if want to understand stuff about `spec` deployment

	k explain Deployment.spec

Labels help you find things and group things

	labels:
		carlos: santana

This is fine but when there are 50 labels it's better to group labels under meaningful naming

---
List deployments

	k get deployments	
	> NAME            READY   UP-TO-DATE   AVAILABLE   AGE
	  my-deployment   0/3     3            0           5s
	k api-resources | grep Deployment
	> deployments                       deploy       apps                           true         *Deployment*
	k get deployments -- show-labels
	> NAME            READY   UP-TO-DATE   AVAILABLE   AGE    LABELS
	  my-deployment   3/3     3            3           4m6s   app=nginx
	k get pods --show-labels
	> NAME                            READY   STATUS    RESTARTS   AGE     LABELS
	  my-deployment-d7c8bbd5c-f6rpt   1/1     Running   0          3m14s   app=nginx,pod-template-hash=d7c8bbd5c
	  my-deployment-d7c8bbd5c-qnmhb   1/1     Running   0          3m14s   app=nginx,pod-template-hash=d7c8bbd5c
	  my-deployment-d7c8bbd5c-rzvhz   1/1     Running   0          3m14s   app=nginx,pod-template-hash=d7c8bbd5c

Note: `app=nginx` shows as *a* label

	k get deploy,rs,pods -l app=nginx
	> NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
	  deployment.apps/my-deployment   3/3     3            3           6m6s

	  NAME                                      DESIRED   CURRENT   READY   AGE
	  replicaset.apps/my-deployment-d7c8bbd5c   3         3         3       6m6s

	  NAME                                READY   STATUS    RESTARTS   AGE
	  pod/my-deployment-d7c8bbd5c-f6rpt   1/1     Running   0          6m6s
	  pod/my-deployment-d7c8bbd5c-qnmhb   1/1     Running   0          6m6s
	  pod/my-deployment-d7c8bbd5c-rzvhz   1/1     Running   0          6m6s
	
	k delete pod/my-deployment-d7c8bbd5c-f6rpt

If you delete a replicaset...

	k delete replicaset.apps/my-deployment-d7c8bbd5c
	k get deploy,rs,pods -l app=nginx
	> NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
	  deployment.apps/my-deployment   3/3     3            3           8m19s

	  NAME                                      DESIRED   CURRENT   READY   AGE
	  replicaset.apps/my-deployment-d7c8bbd5c   3         3         3       9s

	  NAME                                READY   STATUS    RESTARTS   AGE
	  pod/my-deployment-d7c8bbd5c-84qln   1/1     Running   0          9s
	  pod/my-deployment-d7c8bbd5c-ch9kz   1/1     Running   0          9s
	  pod/my-deployment-d7c8bbd5c-nx9z4   1/1     Running   0          9s

The replicaset will get recreated

## Deployments rolling updates and rollback

### Using foo4.yaml

Carlos using split screen w/ 2 terminals
Leave  `watch` command in a separate terminal to monitor deployments, rs, pods

	watch -d "kubectl get deploy,rs,pods
	k set image deployment/my-deployment nginx=nginx:1.16.1 --record
	k set image deployment/my-deployment nginx=nginx:1.16.0 --record

The first `nginx` in `nginx=nginx:1.16.1` refers to the container name

Each `set image` creates an additional new replicaset and new containers
Then, the replica goes away

Carlos thinks it may not work without the `--record`

	kubectl rollout history deployment my-deployment
	> deployment.apps/my-deployment
	  REVISION CHANGE-CAUSE
	  1        <none>
	  2        kubectl set image deployment/my-deployment nginx=nginx:1.16.1 --record=true
	  3        kubectl set image deployment/my-deployment nginx=nginx:1.16.0 --record=true

	k get pods -o yaml | grep image
    > - image: bitnami/nginx:1.16.0
        imagePullPolicy: IfNotPresent
        image: bitnami/nginx:1.16.0
        imageID: docker-pullable://bitnami/nginx@sha256:32a79fce160c3e0688a74e9050b377b223d09bee6e8f7149e7cbc7a6356e5981
      - image: bitnami/nginx:1.16.0
        imagePullPolicy: IfNotPresent
        image: bitnami/nginx:1.16.0
        imageID: docker-pullable://bitnami/nginx@sha256:32a79fce160c3e0688a74e9050b377b223d09bee6e8f7149e7cbc7a6356e5981
      - image: bitnami/nginx:1.16.0
        imagePullPolicy: IfNotPresent
        image: bitnami/nginx:1.16.0
        imageID: docker-pullable://bitnami/nginx@sha256:32a79fce160c3e0688a74e9050b377b223d09bee6e8f7149e7cbc7a6356e5981

	k get pods
	> NAME                            READY   STATUS    RESTARTS   AGE
	  my-deployment-d7c8bbd5c-84qln   1/1     Running   0          14m
	  my-deployment-d7c8bbd5c-ch9kz   1/1     Running   0          14m
	  my-deployment-d7c8bbd5c-nx9z4   1/1     Running   0          14m

## WASdev

[https://github.com/WASdev/sample.plantsbywebsphere](https://github.com/WASdev/sample.plantsbywebsphere)

	git clone git@github.com:WASdev/sample.plantsbywebsphere.git
	gradle tasks
	
"When you have a gradle, it's often nice to look at the tasks"
It'll show you builds, tasks, etc.
Note the **Liberty tasks**

sidecar perks:
* security
* ingress/egress
* getting to and out of a service
* _ALL_ without having to write the code

Java EE:
.war files - web archive
.ear files - enterprise archive; whole big file

Monolith:
business-logic layer
some other layer
web layer
MVC

giant file for Sebastian, which is why microservices :+1:

How many microservices do we need?
Based on the number of people
^ good ballpark, rule of thumb : 2 pizza team

### IBM WebSphere Liberty

main point is that it was fast, at the time
backward compatible

the reason why it was slow because it was so backward compatible, having all the older versions
unlike a pod, which starts up really fast when it goes down

	gradle libertyStart

* install liberty
* compile java
* liberty start
18 seconds

Going to containerize plantsbywebsphere, which will require
* Dockerfile
* .war file from docker image 
	* which we will get from IBM's dockerhub

TWAS = Traditional WebSphere

	if client using websphere:
		99% of the time, run liberty
	
example of lift and shift:
put war file into container

## Jobs and CronJobs

### Using foo5.yaml

	k apply -f foo.yaml
	k get jobs
	k get pods
	> NAME                            READY   STATUS              RESTARTS   AGE
	  my-deployment-d7c8bbd5c-84qln   1/1     Running             0          150m
	  my-deployment-d7c8bbd5c-ch9kz   1/1     Running             0          150m
	  my-deployment-d7c8bbd5c-nx9z4   1/1     Running             0          150m
	  pi-ff2bx                        0/1     ContainerCreating   0          8s
	k get pods -w
	> NAME                            READY   STATUS      RESTARTS   AGE
	  my-deployment-d7c8bbd5c-84qln   1/1     Running     0          152m
	  my-deployment-d7c8bbd5c-ch9kz   1/1     Running     0          152m
	  my-deployment-d7c8bbd5c-nx9z4   1/1     Running     0          152m
	  pi-ff2bx                        0/1     Completed   0          2m2s
	k describe job pi

When running in parallel:

### foo6.yaml

	spec:
		parallelism: 2
		completions: 3

Want 3 jobs to be done but can run 2 at a time.

### CronJob:

#### foo7.yaml

	schedule: "*/1 * * * *"

schedule: every minute

	k get pods
	k get cronjob -o wide
---
#### Q&A

`parallelism` and `completions` work in job spec of CronJob as well

---
To watch logs of the CronJob:

	stern .

## Liberty_Start.pptx

PowerPoint slide from Matt in Slack `#cpat-cn-bc-rtp`

Middleware performance: IBM > Oracle
Startup Time & Footprint (Memory): Liberty > Other Java EE
Throughput : TWAS >~ Liberty > Other Java EE

## Lab
[Rolling Updates](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab6/)

	k set image deployment/jedi-deployment jedi-ws-bitnamy/nginx:1.10.1 --record

Check the progress of the rolling update:

	k rollout status deployment/jedi-deployment

In another terminal window

	k get pods -w

Get a list of previous revisions

	k rollout history deployment/jedi-deployment

Undo the last revision

	k rollout undo deployment/jedi-deployment

[CronJobs](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab7/)


## Services & Networking
[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/services-networking/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/services-networking/)

Service object works in conjunction with pods to get endpoints
Deployment easiest way to show an example of service w/ pod:
Service creates the endpoint 

### Using foo8.yaml

It is beneficial to have a name with a port to identify the specific port number:

	ports:
		- containerPort: 8080
		  name: http
port can be changed all the time, so having a name is a good reference

Carlos starts analyzing the service alone in foo8.yaml

	apiVersion: v1
	kind: Service
	metadata:
	  name: my-service
	spec:
	  selector:
		  app: nginx
	  ports:
		  - name: http
		    port:  80
		    targetPort: http
[//]: #(Commands)
	k get pods --show-labels

To check `spec` documentation:

	k explain Service.spec
	
[//]: #(ContinueCommands)

	k get pods -l app=nginx -o wide
	k get pods -l app=nginx -o wide --show-labels
	
	k get pods
	k get services
	> NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
	  kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   25h
	  my-service   ClusterIP   10.100.233.180   <none>        80/TCP    8m4s
	k describe svc my-service
	> Name:              my-service
	  Namespace:         default
	  Labels:            <none>
	  Annotations:       kubectl.kubernetes.io/last-applied-configuration:
	                       {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"my-service","namespace":"default"},"spec":{"ports":[{"name":"http...
	  Selector:          app=nginx
	  Type:              ClusterIP
	  IP:                10.100.233.180
	  Port:              http  80/TCP
	  TargetPort:        http/TCP
	  Endpoints:         172.17.0.10:8080,172.17.0.11:8080,172.17.0.5:8080
	  Session Affinity:  None
	  Events:            <none>
	k get endpoints -o wide
	> NAME         ENDPOINTS                                           AGE
	  kubernetes   192.168.64.5:8443                                   25h
	  my-service   172.17.0.10:8080,172.17.0.11:8080,172.17.0.5:8080   12m
	k get endpoints my-service -o wide
	> NAME         ENDPOINTS                                           AGE
	  my-service   172.17.0.10:8080,172.17.0.11:8080,172.17.0.5:8080   12m
	k get ep my-service -o yaml
	> apiVersion: v1
	  kind: Endpoints
	  metadata:
	    annotations:
	      endpoints.kubernetes.io/last-change-trigger-time: "2020-02-05T20:16:18Z"
	    creationTimestamp: "2020-02-05T20:16:14Z"
	    name: my-service
	    namespace: default
	    resourceVersion: "37980"
	    selfLink: /api/v1/namespaces/default/endpoints/my-service
	    uid: 848a6d72-c60b-4288-b680-a93333cfc7eb
	  subsets:
	  - addresses:
	    - ip: 172.17.0.10
	      nodeName: minikube
	      targetRef:
	        kind: Pod
	      name: my-deployment-cbf45cf84-fkvzd
	      namespace: default
	      resourceVersion: "37979"
	      uid: c4d60553-5fe9-45f3-a97e-6bb1ef68ea87
	    - ip: 172.17.0.11
		  nodeName: minikube																																																																																																																																							    
		  targetRef:																																																																																																																																							      
		    kind: Pod
		    name: my-deployment-cbf45cf84-ng48c
		    namespace: default
		    resourceVersion: "37951"
		    uid: 08bd4ca2-6919-4c7b-beb2-59b4816f1413
		- ip: 172.17.0.5
		  nodeName: minikube
		  targetRef:
		    kind: Pod																																																																																																																																	      
		    name: my-deployment-cbf45cf84-gxmhx																																																																																																																																						      
		    namespace: default																																																																																																																																				      
		    resourceVersion: "37924"																																																																																																																																      
		    uid: 7e13a76b-21fc-4cc0-adb8-5656cd42b150																																																																																																																																								  
	    ports:																																																																																																																																							  
	    - name: http																																																																																																																																							   
	    - port: 8080																																																																																																																																							    
	    - protocol: TCP

### Kubernetes NodePort
	spec:
		type: NodePort

NodePort randomly assigns a port number

	curl <IP ADDRESS>
	curl http://192.168.64.34:31211

when ran with `curl` the port is assigned to 

## Ingress

	minikube addons enable ingress
	> ✅  ingress was successfully enabled
	minikube config view
	> - metrics-server: true
	  - cpus: 4
	  - ingress: true
	  - kubernetes-version: v1.16.6
	  - memory: 4096
	k get pods -n kube-system | grep ingress
	> nginx-ingress-controller-6fc5bcc8c9-2h79g   1/1     Running   0          75s
	k run web --image=bitnami/nginx --port=8080
	> kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
	  deployment.apps/web created
	k get deploy
	> NAME              READY   UP-TO-DATE   AVAILABLE   AGE
	  web               1/1     1            1           8s
	k expose deployment web --target-port=8080 --type=NodePort
	> service/web exposed
	k get svc web
	> NAME   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	  web    NodePort   10.109.199.105   <none>        8080:31009/TCP   11s

`8080:31009/TCP` randomly generated port number
can be placed in with curl again

	k describe service web
	> Name:                     web
	  Namespace:                default
	  Labels:                   run=web
	  Annotations:              <none>
	  Selector:                 run=web
	  Type:                     NodePort
	  IP:                       10.109.199.105
	  Port:                     <unset>  8080/TCP
	  TargetPort:               8080/TCP
	  NodePort:                 <unset>  31009/TCP
	  Endpoints:                172.17.0.12:8080
	  Session Affinity:         None
	  External Traffic Policy:  Cluster
	  Events:                   <none>

Ingress needs to be in the same webspace as `web`

	stern ingress 0n kube-system
	k get ingress
	k describe ingress example-ingress

Configure Ingress `rules` for traffic

easy way:
exposes every service, not secure but it's easy to run nginx within

	spec:
		rules:
			- http:
				paths:
					- path: /
					  backend:
						  serviceName: web
						  servicePort: 8080

## Lab
[Services](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab8/)