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
``build/libs`` folder, but we want to containerise it so that it can be run within a container.

## Dockerisation

The docker file for the application is in ``src/main/docker``. We need a container with jdk to run our springboot application.
There are many images with jdk 8 publicly available already but just to get the end to end experience I built a minimal
container based on alpine linux with Jdk 8 based on image published by Anastas Dancha's <anapsix@random.io>.

The ``Dockerfile`` for the java8 image is included within
``docker/alpine-java8`` in the project.

Now build image and push it to the docker registry.

```
cd src/docker/alpine-java8
docker build -t achalise/alpine-java8 
docker push achalise/alpine-java8
```

The ``Dckerfile`` for the application located in ``src/main/docker`` uses the image published eariler as base image.

```$xslt
FROM achalise/alpine-java8:latest
VOLUME /tmp
ADD springboot-kubernetes-0.0.1-SNAPSHOT.jar app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -Dspring.data.mongodb.host=$MONGO_URI \
-jar /app.jar" ]
```

Take note that the ``ENTRYPOINT`` in ``Dockerfile`` uses ``MONGO_URI`` environment variable for the MongoDB hostname.
We build docker image ``achalise/spring-boot-service`` for our app using gradle task ``buildDockerImage`` available from ``se.transmode.gradle:gradle-docker:1.2``
which is included in the build classpath.
``docker push achalise/spring-boot-service``