# Kubernetes
Interesting link: https://thenewstack.io/taking-closer-look-kubernetes-part-two/
## Microservices
Reduce the size of applications to the size of a microservice that allows to deploy them instantly on containers.
Microservices increases the speed of development and are useful in continuous development. Each team can focus on their service, developing at their own speed. 
However they should try to maintain the interface as invariant as possible.
To compose services, use Docker compose, or Kubernetes configuration file.
Micro services is a design pattern that allows to write applications that are easy to deploy, modular, and scale independantly. 

## JSON Web Tokens
It is a compact way to send data between parties as a JSON object. It is mostly used for authentification: an token is created by a server and is used by the client to be able to retrieve information from a server after being authentified.

## Docker
A container is a running instance of an image. An image is used to boot up a new container, and it is self sufficient in its own to represent the dependency of the container.
* Show all images downloaded: ```docker images```
* Install images: ```docker pull image_name_:version```
* Run an image: ```docker run -d image_name:version```
* Check if a container is running ```docker ps```
* Get information about an container: ```docker inspect container_name```
* Stop a container ```docker stop *```
* Delete file left of a running container ```docker rm *```

### Build our own images with Dockerfile
Simple example of a Docker file:
```
# Which image to use as default, usually alpine
FROM base_image 
ADD file /path/in/container
# Add a program to launch when starting the container
ENTRYPOINT ["hello"]
```
Then ```docker build --file /path/dockerfile```

Finally, you can tag your container, and push them to a Docker Hub, so that it will be shared with the community.

## Kubernetes
Time to thing about scaling up, with more than one machine.
Serverless computing: as starting container is very fast compared to a VM, you can start a container to run a job for five minutes, and then shut it down, and reuse it when you want.
Kubernetes look at container at a higher level, as clusters become as simple as managing a single logical machine.
In Kubernetes all containers run in pods. 
* To see all the pods, use ```kubectl get pods```
* To run a particular docker image ```kubectl run name --image=docker_image```
* To expose the pods to the Internet, you do ```kubectl expose deployments name```
* To get info about the services running ```kubectl get services```

### Pods
Pods represent a logical application of one or more containers.
Pods can be created with YAML configuration file, specifying the Docker containers, the ressource allocated on the pod for the container, the exposed port...
* To get inside a pod ```kubectl exec name /bin/bash```

### Monitoring pods
Kubelet manages containers that have been created by Kubernetes and restart them in case one of them is done. As pod get restart over and over, their Ip address might changes between restart. To allows pod to be able to still comunicate, pods relies on Services. The pods that a Service exposed are based on a set of labels, hence if pods have the same labels, they are automatically picked up. A Service can whether give IP address to each pods, or load balance the traffic to pods within it.
The idea is to tag pods with label, and create a Service with a configuration file where you define which label will be used to retrieve the pods to used for the Service.

### Create Deployements
Behind the scenes, Deployements manage replica sets. Each Deployment is mapped to one active replica set.

## Updating code
Code can be updated (new images), then the Service can manage to rollout this change to the production by creating and removing one at a time new pods with the recent changes versus old pods. At no point, there are no pods running, the transition goes smootly. This update is done automatically if you update any image in the Deployment configuration file. 
