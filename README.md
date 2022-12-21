# WildFly s2i pipeline for Openshift

A pipeline to produce an application image using the WildFly s2i builder and runtime image. Image built using Buildah.
The pipeline optionally creates a deployment.

## Pre-requisites

*  Logged into an OpenShift cluster (such as OpenShift Sandbox).

## Install the WildFly s2i task and pipeline

* ``oc create --filename wildfly-s2i-build-task.yaml``
* ``oc create --filename wildfly-s2i-build-pipeline.yaml``

## Create a PVC (Maven cache and shared data between tasks)

* ``oc create --filename  pvc-sources-maven.yaml``

Grant all access to user (required to create POD, services and pipelines from event listener).
* ``oc create --filename  rbac.yaml``

Example: Build an image, create a POD and service

* Build: ``oc create --filename examples/test-app-pipeline-run.yaml``
* In OpenShift console you can monitor the started pipeline run (from ``Pipelines/Pipelines/PipelineRuns`` ).


Event listener example: Start the pipeline from an external event

Install tekton triggers

* ``kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml``
* ``kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml``

Create the Event listener (a service named el-my-listener is created)

* ``kubectl apply --filename listener-demo/event-listener.yaml``
* ``kubectl apply --filename listener-demo/pipeline-template.yaml``
* Enable port forwarding to send event to the listener: ``kubectl port-forward service/el-my-listener 8080 &``

Send the event

* ``curl -X POST --data {} 127.0.0.1:8080``
* In the tekton dashboard, a new pipeline run is started, you can monitor its state.
