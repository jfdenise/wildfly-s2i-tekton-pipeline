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

3) Create the application pipeline

The output of this pipeline is an application image deployed with Helm Charts

* ``oc create --filename f2f/app-run.yaml``
* Then monitor the progress in the Openshift console (Pipelines)

4) Access the service

* curl https://$(oc get route helloworld-from-pipeline --template='{{ .spec.host }}')/HelloWorld


