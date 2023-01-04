# WildFly s2i pipelines for Openshift

We are defining 3 Tekton pipelines:

* A pipeline that does a full S2I build. It provisions the WildFly server, deploys the application into it, generates an ImageStream
and (optionally) creates a deployment and exposes the service.

* A pipeline that provisions a WildFly server to produce a custom WildFly S2I builder imageStream 
that can then be used to do an S2I build of an application.

* A pipeline that build an application using a custom WildFly S2I builder image (produced by the previous pipeline). 
In such case, no WildFly server provisioning occurs. Simple copy of the war file is operated and a runtime image is produced. 
Optionally a deployment is created and the service is exposed.

These 3 pipelines use the `wildfly-s2i-build-task` Tekton Task, `git-clone` Tekton ClusterTask and the `buildah` Tekton ClusterTask, 

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

* Build: ``oc create --filename runs/test-app-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the pipeline run, the deployment `test-app` is scaled to 1 and the service is exposed.

## Custom WildFly s2i builder build Tekton pipeline example

In this example we are producing a WildFly S2I builder allowing to build and run 
applications requiring the `jaxrs-server` and `ejb` Galleon layers.

### Create the pipeline

``oc create --filename wildfly-s2i-builder-pipeline.yaml``

### Create a Pipeline run

* Build: ``oc create --filename runs/jaxrs-ejb-builder-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the run, the ImageStream `jaxrs-ejb-server:latest` is created. This image Stream will be used in the next example 
to produce an application image.

## Application build with custom WildFly s2i builder Tekton pipeline example

In this example we are re-using the `jaxrs-ejb-server` custom S2I builder to produce an application image 
and scale / expose the deployment..

### Create the pipeline

``oc create --filename wildfly-s2i-build-app-pipeline.yaml``

### Create a Pipeline run

* Build: ``oc create --filename runs/app-from-jaxrs-ejb-builder-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).

At the end of the pipeline run, the deployment `helloworld` is scaled to 1 and the service is exposed.


## Event listener example: Start the pipeline from an external event

### Create the Template and Event listener (a service named el-test-app-event-listener is created and exposed)

* ``oc apply --filename github-listener/pipeline-template.yaml``
* ``oc apply --filename github-listener/event-listener.yaml``
* ``oc expose service/el-test-app-event-listener``

### Create the webhook

* ``echo $(oc  get route el-test-app-event-listener --template='http://{{.spec.host}}')``
* Copy the URL printed in the console.
* Add a webhook in your git repository

### Push a new commit

`` git commit -m "empty-commit" --allow-empty && git push origin test-openshift-pipeline ``