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
```
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
		classpath('se.transmode.gradle:gradle-docker:1.2')
	}
```
Then log into docker, with your credentials after signing up on [docker.io](docker.io), 
```
docker login
```
And execute the following command to push image to docker registry.

```docker push achalise/spring-boot-service```

## Deploying on Kubernetes

Now that we have image of our application available in the docker registry, we can deploy it in a kubernetes cluster.
We will also set up a node for mongoDB to be used as backend by our application.

Start local kubernetes cluster with the following command:

``` minikube start```

We can then launch dashboard for the cluster:

```minikube dashboard```

Next, set up a MongoDB node. The replication controller and service configuration to set up MongoDB pod are included
in the project within
``k8s/mongodb``

First create a MongoDB service which will be used our application running in a different POD to connect to the mongoDB 
server.

```
kubectl create -f mongo-service.yml
```
The above command will create service with the name ``mongo-service`` which is used by our app to access mongoDB database.
In ``mongo-service.yml``, following entry assigns name to the service:

```
metadata:
  labels:
    name: mongo
  name: mongo-service

```
And out application uses ``mongo-service`` as value for the environment variable ``MONGO_URI`` as described in 
``deployment.yml``:

```
        env:
            - name: MONGO_URI
              value: mongo-service
```
Now create MongoDB instance that actually runs the database.

```kubectl create -f mongo-controller.yml```

Next create deployment of our application in the cluster. The config files are located in ``k8s/``

```kubectl create -f deployment.yml```

Above command will create two pods, now create a service to connect to these pods.

```kubectl create -f service.yml```

You can see description of the service with

```kubectl describe service spring-boot-service```

Now get the exact address of the service with

```minikube service spring-boot-service```

which will launch browser and point to the endpoint. For e.g. in my case,

```
curl http://192.168.99.101:30864/user =>
[{"id":"58bcd7ad5908010005cce257","firstName":"Arun","lastName":null,"email":null,"address":{"street1":null,"street2":null,"town":null,"postcode":null,"state":null}}]

```