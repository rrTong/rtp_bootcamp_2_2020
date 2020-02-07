# Thursday 2/6

## Ingress - Type LoadBalancer

[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/services-networking](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/services-networking)

Talked about yesterday: unique port number, external name or ip
Load Balancer is another type aside from `NodePort` and `ClusterIP`

In Service...
	
	spec:
		type: LoadBalancer

Once configured, anyone can access the `LoadBalancer`
the service/endpoint will constantly update the `LoadBalancer`

Just changing the type to `LoadBalancer` isn't enough, need to run routing
Run `minikube tunnel` in a separate terminal

	minikube tunnel
	k get svc my-service
	> NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
	  my-service   LoadBalancer   10.111.140.66   <pending>     80:30825/TCP   26s
	k get svc my-service
	> NAME         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
	  my-service   LoadBalancer   10.111.140.66   10.111.140.66   80:30825/TCP   45s

Once **EXTERNAL-IP** of the `LoadBalancer` is copied into a browser, can access externally

## Volumes

[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/state-persistence/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/state-persistence/)

### Using foo2.yaml
	
	k apply -f foo.yaml
	k get pods
	> NAME                             READY   STATUS              RESTARTS   AGE
	  my-pod                           0/1     ContainerCreating   0          3s

Can work within this pod
	
	k exec -it my-pod bash

### Using foo3.yaml

	k apply -f foo.yaml
	k get pods
	> NAME                             READY   STATUS    RESTARTS   AGE
	  test-pd                          1/1     Running   0          2s
	k exec -it test-pd bash
	I have no name!@test-pd:/app$ sudo echo "hello from container" > bar.txt

#### Had `Permission denied` issues

	minikube ssh

Have `sudo` on for the rest of the session
	
	sudo su

Permissions to read/write

	chmod -R 777

## PersistentVolumes and PersistentVolumeClaims

Abbreviated as PV and PVC
PV&PVC team that works with pod, application to have **requirements**

The goal is to give people access to read or write to folders
PVC declares the requirement, PV is the volume itself
PVC is satisfied by the PV

The volume is persistent, persistent as in the volume does not go away

### Access Modes

-   ReadWriteOnce – the volume can be mounted as read-write by a single node
-   ReadOnlyMany – the volume can be mounted read-only by many nodes
-   ReadWriteMany – the volume can be mounted as read-write by many nodes

[more info](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

### Using foo4.yaml

	accessModes:
		- ReadWriteOnce

only need 100Mi storage, don't need that much

	k get pv
	> NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
	  my-pv   128Mi      RWO            Retain           Available           local-storage            3s

### Using foo5.yaml

	spec:
		storageClassName: local-storage
	...
	accessModes:
		- ReadWriteOnce
	...
	requests:
		storage: 100Mi

[//]: #(break)

	k get pvc
	> NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
	  my-pvc   Bound    my-pv    128Mi      RWO            local-storage   2s

PVC is tied to a PV based on `capacity.storage` and `requests.storage`
the PVC must be removed unless the PV will not be deleted (even if `k delete pv`)

## Lab

[Persistent Volumes](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/kubernetes/activities/labs/lab10/)

