# WildFly s2i pipeline for Openshift

A pipeline to produce an application image using the WildFly s2i builder and runtime image. Image built using Buildah.
The pipeline deploys the application and exposes a route to it. Deployment can be disabled.

## Pre-requisites

*  Logged into an OpenShift cluster (such as OpenShift Sandbox).

## Create the WildFly s2i task and pipeline

* ``oc create --filename wildfly-s2i-build-task.yaml``
* ``oc create --filename wildfly-s2i-build-pipeline.yaml``

## Create a PersistentVolumeClaim (Maven cache and shared data between tasks)

* ``oc create --filename  pvc-sources-maven.yaml``

## PipelineRun example

* Build: ``oc create --filename examples/test-app-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).


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