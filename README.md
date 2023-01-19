# WildFly s2i pipelines for Openshift

We are defining 3 Tekton pipelines:

* A pipeline that does a full S2I build. It provisions the WildFly server, deploys the application into it, generates an ImageStream
and (optionally) deploys the image using WildFly Helm charts.

* A pipeline that provisions a WildFly server to produce a custom WildFly S2I builder and runtime imageStreams
that can then be used to do an S2I build of an application.

* A pipeline that build an application using a custom WildFly S2I builder and runtime images (produced by the previous pipeline). 
In such case, no WildFly server provisioning occurs. The execution of the wildfly-maven-plugin is skip. 
Simple copy of the war file is operated and a runtime image is produced. 
Optionally deploys the image using WildFly Helm charts.

These 3 pipelines use the `wildfly-s2i-build-task` Tekton Task, `git-clone` Tekton ClusterTask and the `buildah` Tekton ClusterTask.

The deployment operated from the pipelines use the `openshift-client` and `helm-upgrade-from-repo` Tekton ClusterTask.

## Pre-requisites

*  Logged into an OpenShift cluster (such as OpenShift Sandbox).

## Create the WildFly s2i task

* ``oc create --filename wildfly-s2i-build-task.yaml``

## Create a PersistentVolumeClaim (Maven cache and shared data between tasks)

* ``oc create --filename  pvc-sources-maven.yaml``

## Full s2i build Tekton pipeline example

This pipeline builds the WildFly server, the application, the application image then deploys the application image.

### Create the pipeline

``oc create --filename wildfly-s2i-build-pipeline.yaml``

### Create a Pipeline run

* Build: ``oc create --filename runs/full-build-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the pipeline run, the deployment `microprofile-full` is scaled to 1 and the service is exposed.

## Custom WildFly s2i builder build Tekton pipeline example

In this example we are producing a custom WildFly S2I builder and runtime images allowing to build and run 
applications that require the `jaxrs-server` and `microprofile-config` Galleon layers.

### Create the pipeline

``oc create --filename wildfly-s2i-builder-pipeline.yaml``

### Create a Pipeline run

* Build: ``oc create --filename runs/jaxrs-microprofile-builder-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the run, the ImageStream `jaxrs-microprofile-server-builder:latest` and `jaxrs-microprofile-server:latest` are created. 
These ImageStreams will be used in the next example to produce an application image.

## Application build with custom WildFly s2i builder and runtime Tekton pipeline example

In this example we are re-using the `jaxrs-microprofile-server-builder` and `jaxrs-microprofile-server` ImageStreams to produce an application image.
Helm charts is then used to deploy the image.

### Create the pipeline

``oc create --filename wildfly-s2i-build-app-pipeline.yaml``

### Create a Pipeline run

* Build: ``oc create --filename runs/app-from-jaxrs-microprofile-builder-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the pipeline run, the image `microprofile` is created, scaled to 1 and the service is exposed.


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
* Add a webhook in your git repository fork (Settings/Webhooks) using this URL.

### Push a new commit

In your cloned repository push an empty commit to activate the webhook and start a pipeline run.

`` git commit -m "empty-commit" --allow-empty && git push origin test-openshift-pipeline ``

A new pipeline run is created, a new image is created, a new revision of the Helm Charts is created, the deployment is re-deployed.