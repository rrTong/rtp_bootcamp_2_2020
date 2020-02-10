# Friday 2/7

## 1-1 with Kirk, missed first hour

## OpenShift/Tekton Deployment

[https://github.com/csantanapr/cloudnative_sample_app_deploy](https://github.com/csantanapr/cloudnative_sample_app_deploy)
[https://github.com/csantanapr/cloudnative_sample_app](https://github.com/csantanapr/cloudnative_sample_app)

`demo` = name of task

	oc apply -f my-task.yaml
	tkn task ls
	
	tkn task start demo

To see service accounts

	oc get sa

Service accounts run the pod, in this case run the task

	tkn task start demo -s pipeline

To monitor logs in the background in a separate terminal

	stern .

[//]: #(break)

	oc get pods

	tkn pipeline start pipeline-git-push -s pipeline
	tkn pipelinerun logs pipeline

## CICD Project
[https://pages.github.ibm.com/CASE/cloudnative-bootcamp/project-cicd/](https://pages.github.ibm.com/CASE/cloudnative-bootcamp/project-cicd/)
[IBM Cloud Cluster](https://cloud.ibm.com/kubernetes/clusters/bopdvafd0rnhhaa0b4dg/overview?region=us-south&resourceGroup=c9e1d3f5c2e4430294b10f0bce0159f1)