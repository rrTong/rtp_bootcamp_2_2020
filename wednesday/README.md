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

## Notes
[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/pod-design/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/pod-design/)

## # Labels, Selectors, and Annotations

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