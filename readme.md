#Spring Boot  with MongoDB in Kubernetes Cluster

This project is an exercise to deploy spring boot rest application with a mongodb in the backend on a local 
kubernetes cluster. 

## Prerequisites

Following tools are needed to create container image of our application and deploy it 
on a locally running kubernetes cluster.
*  [Docker](https://docs.docker.com/engine/installation/) to create docker images that will be deployed on 
kubernetes cluster
* [Minikube](https://kubernetes.io/docs/getting-started-guides/minikube/#installation) to run kubernetes locally
* [Kubectl](https://kubernetes.io/docs/user-guide/prereqs/) to interact with the kubernetes cluster via command line

## SpringBoot application

The application is a simple REST application to list all users or a specific user
from a user collection stored in a MongoDB backend. The gradle build creates a spring boot fat jar ``springboot-kubernetes-0.0.1-SNAPSHOT``
in ``build/libs`` folder. The application can be run with ``java -jar springboot-kubernetes-0.0.1-SNAPSHOT.jar`` from
``build/libs`` folder, but we want to containerise it so that it can be run on a container.
 