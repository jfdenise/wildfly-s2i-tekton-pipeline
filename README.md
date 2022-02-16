# WildFly s2i V2 pipeline

A pipeline to produce an application image using the WildFly s2i builder and runtime image. Image built using Kaniko.

Using minikube v1.25.1, push image to local registry

./minikube start
minikube addons enable registry
minikube addons enable registry-aliases

Install tekton:
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.30.0/release.yaml

Install tasks from catalog:
kubectl create --filename https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.5/git-clone.yaml
kubectl create --filename https://raw.githubusercontent.com/tektoncd/catalog/main/task/kaniko/0.5/kaniko.yaml

Install WildFly s2i task and pipeline

kubectl create --filename wildfly-s2i-build-task.yaml
kubectl create --filename wildfly-s2i-build-pipeline.yaml

Create PVC (Maven cache and shared data between tasks)

kubectl create --filename  pvc-sources-maven.yaml

Example 1:

* V2 build: kubectl create --filename examples/test-app-build.yaml
* Run the image: ./kubectl create deployment test-app --image=registry.minikube/wildfly/sample-test-app-v2
* Expose the port: ./kubectl expose deployment test-app --type=LoadBalancer --port=8080
* Access the application: ./minikube service test-app

Example 2:

* V2 build with legacy quickstart: kubectl create --filename examples/qs-rest-client-server-build.yaml
* Run the image: ./kubectl create deployment qs-rest-client-server --image=registry.minikube/wildfly/rest-client-server-qs
* Expose the port: ./kubectl expose deployment qs-rest-client-server --type=LoadBalancer --port=8080
* Access the application: ./minikube service qs-rest-client-server
