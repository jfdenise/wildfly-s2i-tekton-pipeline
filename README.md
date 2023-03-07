# WildFly s2i pipeline and task for Openshift

We are defining a pipeline that does a full S2I build. It provisions the WildFly server, deploys the application into it, generates an ImageStream
and (optionally) deploys the image using WildFly Helm charts.

This pipeline uses the `wildfly-s2i-build-task` Tekton Task, `git-clone` Tekton ClusterTask and the `buildah` Tekton ClusterTask.

The deployment operated from the pipelines use the `openshift-client` and `helm-upgrade-from-repo` Tekton ClusterTask.

## WildFly s2i build task

This task output a Dockerfile and a dockerContext results for the next task to build an application image. Caching can be enabled.
When caching is enabled, during the first execution, the provisioned server is cached and a server-only image DockerFile is also generated.
This server-only image is the from image of the application image.
When the cache is already populated, provisioning of the server is disabled and the application image DockerFile is only generated.
The server-only image from the previous step is automatically re-used.


## Pre-requisites

*  Logged into an OpenShift cluster (such as OpenShift Sandbox).

## Create the WildFly s2i task

* ``oc create --filename wildfly-s2i-build-task.yaml``

## Create a PersistentVolumeClaim (Maven cache and shared data between tasks)

* ``oc create --filename  pvc-sources-maven.yaml``

## Full s2i build Tekton pipeline example

This pipeline builds the WildFly server, the application, the server image and the application image then deploys the application image.

### Create the pipeline

``oc create --filename wildfly-s2i-build-pipeline.yaml``

### Create a Pipeline run

* Build: ``oc create --filename runs/full-build-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the pipeline run, the deployment `microprofile` is scaled to 1 and the service is exposed.

### Create a second Pipeline run

* Build: ``oc create --filename runs/full-build-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

This time you will notice that the task to build/push the server image is disabled.

At the end of the pipeline run, the deployment `microprofile` is scaled to 1 and the service is exposed.

## Event listener example: Start the pipeline from an external event

NOTE: We are using a fork of `github.com/jfdenise/wildfly-s2i-tekton-pipeline` repository. If not already done, fork this repository and clone it.
Create a branch named `test-openshift-pipeline`.

### Create the Template and Event listener (a service named el-test-app-event-listener is created and exposed)

* ``oc create --filename github-listener/pipeline-template.yaml``
* ``oc create --filename github-listener/event-listener.yaml``
* ``oc expose service/el-test-app-event-listener``

### Create the webhook

* ``echo $(oc  get route el-test-app-event-listener --template='http://{{.spec.host}}')``
* Copy the URL printed in the console.
* Add a webhook in your git repository fork (Settings/Webhooks) using this URL of type `application/json`.

### Push a new commit

In your cloned repository push an empty commit to activate the webhook and start a pipeline run.

`` git commit -m "empty-commit" --allow-empty && git push origin test-openshift-pipeline ``

A new pipeline run is created, a new image is created, a new revision of the Helm Charts is created, the deployment is re-deployed.