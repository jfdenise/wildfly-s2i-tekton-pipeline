# WildFly s2i V2 pipeline

A pipeline to produce an application image using the WildFly s2i builder and runtime image. Image built using Kaniko.
The pipeline optionally creates a deployment.

Using minikube v1.25.1, push image to local registry

* ``minikube start``
* ``minikube addons enable registry``
* ``minikube addons enable registry-aliases``

Install tekton

* ``kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.30.0/release.yaml``
* ``kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml``

Install tasks from catalog

* ``kubectl apply --filename https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.5/git-clone.yaml``
* ``kubectl apply --filename https://raw.githubusercontent.com/tektoncd/catalog/main/task/kaniko/0.5/kaniko.yaml``
* ``kubectl apply --filename https://raw.githubusercontent.com/tektoncd/catalog/main/task/kubernetes-actions/0.2/kubernetes-actions.yaml``

Install WildFly s2i task and pipeline

* ``kubectl create --filename wildfly-s2i-build-task.yaml``
* ``kubectl create --filename wildfly-s2i-build-pipeline.yaml``

Create PVC (Maven cache and shared data between tasks)

* ``kubectl create --filename  pvc-sources-maven.yaml``

Grant all access to user (required to create POD, services and pipelines from event listener).
* ``kubectl create --filename  rbac.yaml``

Enable proxy to access to the Tekton dashboard

* ``kubectl proxy &``
* Access to ``http://localhost:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/#/namespaces/default/pipelines``
* You should see the ``wildfly-s2i-build-pipeline``.

Example 1: Build an image, create a POD and service

* V2 build: ``kubectl create --filename examples/test-app-build.yaml``
* In tekton dashboard you can monitor the started pipeline run (from ``PipelineRuns`` ).
* When done, you can access the application: ``minikube service test-app``

Example 2: Build an image from an existing quickstart, create a POD and service

* V2 build with legacy quickstart: ``kubectl create --filename examples/qs-rest-client-server-build.yaml``
* In tekton dashboard you can monitor the started pipeline run (from ``PipelineRuns`` ).
* When done, you can access the application: ``minikube service qs-rest-client-server``

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
