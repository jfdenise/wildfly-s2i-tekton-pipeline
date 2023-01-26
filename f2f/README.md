# WildFly s2i pipelines for Openshift, F2F workshop

In this workshop you are going to learn how to use Tekton pipeline for an optimized WildFly server build and deployment

## Steps

1) Create the "WildFly build" task needed from out pipelines, the persistent storage (maven cache) and the 2 pipelines.

* ``oc create --filename wildfly-s2i-build-task.yaml``
* ``oc create --filename  pvc-sources-maven.yaml``
* ``oc create --filename wildfly-s2i-builder-pipeline.yaml``
* ``oc create --filename wildfly-s2i-build-app-pipeline.yaml``

2) Create a pipeline run to build the WildFly Builder and runtime images

The output of this pipeline run is 2 imageStreams (the builder and runtime images) that both contain the provisioned WildFly server

* ``oc create --filename f2f/builder-run.yaml``
* Then monitor the progress in the Openshift console (Pipelines)

3) Create the application pipeline run

The output of this pipeline is an application image deployed with Helm Charts

* ``oc create --filename f2f/app-run.yaml``
* Then monitor the progress in the Openshift console (Pipelines)

4) Access the service

* curl https://$(oc get route helloworld-from-pipeline --template='{{ .spec.host }}')/HelloWorld

## Event listener example: Start the pipeline from an external event

NOTE: We are using a fork of `github.com/wildfly/quickstart` repository. If not already done, fork this repository and clone it.
Create a branch named `test-openshift-pipeline`.

### Create the Template and Event listener (a service named el-test-app-event-listener is created and exposed)

* ``oc create --filename f2f/github-listener/pipeline-template.yaml``
* ``oc create --filename f2f/github-listener/event-listener.yaml``
* ``oc expose service/el-test-app-event-listener``

### Create the webhook

* ``echo $(oc  get route el-test-app-event-listener --template='http://{{.spec.host}}')``
* Copy the URL printed in the console.
* Add a webhook in your git repository fork (Settings/Webhooks) using this URL of type `application/json`.

### Push a new commit

In your cloned repository push an empty commit to activate the webhook and start a pipeline run.

`` git commit -m "empty-commit" --allow-empty && git push origin test-openshift-pipeline ``

A new pipeline run is created, a new image is created, a new revision of the Helm Charts is created, the deployment is re-deployed.
