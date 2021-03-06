# IBM-ODM-Kubernetes
IBM Operational Decision Manager on Kubernetes

[![Build Status](https://travis-ci.org/PierreFeillet/IBM-ODM-Kubernetes.svg?branch=master)](https://travis-ci.org/PierreFeillet/IBM-ODM-Kubernetes)


#  Deploy IBM Operational Decision Manager Standard on Kubernetes Cluster

This code demonstrates the deployment of an IBM Operational Decision Manager clustered topology using WAS Liberty on a Bluemix Kubernetes Cluster.

We leverage the ODM Docker material put available on this repository [odm-ondocker](https://github.com/lgrateau/odm-ondocker). It includes Docker files and Docker compose descriptors. In this tutorial we will only use the Docker files to build the ODM runtime images that we will instantiate in the Kubernetes cluster.

![Flow](../images/ODMinKubernetes-Flow.png)

## Included Components
- [IBM ODM](https://www.ibm.com/support/knowledgecenter/SSQP76_8.9.0/welcome/kc_welcome_odmV.html)
- [Kubernetes Cluster](https://console.ng.bluemix.net/docs/containers/cs_ov.html#cs_ov)
- [Bluemix Container Service](https://console.ng.bluemix.net/catalog/?taxonomyNavigation=apps&category=containers)
- [Bluemix DevOps Toolchain Service](https://console.ng.bluemix.net/catalog/services/continuous-delivery)

## Testing
This tutorial has been tested on MacOS.

## Prerequisite

* Create a Kubernetes cluster with either [Minikube](https://kubernetes.io/docs/getting-started-guides/minikube) for local testing, or with [IBM Bluemix Container Service](https://github.com/IBM/container-journey-template) to deploy in cloud. For deploying on Minikube follow the instructions [here](https://github.com/WASdev/sample.microservicebuilder.docs/blob/master/dev_test_local_minikube.md)
* The code in this particular repository is regularly tested against [Kubernetes Cluster from Bluemix Container Service](https://console.ng.bluemix.net/docs/containers/cs_ov.html#cs_ov) using Travis.
* Install a Git client to obtain the sample code.
* Install [Maven](https://maven.apache.org/download.cgi) and a Java 8 JDK.
* Install a [Docker](https://docs.docker.com/engine/installation/) engine.


## Deploy to Kubernetes Cluster from Bluemix
If you want to deploy directly to Bluemix, click on 'Deploy to Bluemix' button below to create a Bluemix DevOps service toolchain and pipeline for deploying the Java microservices using MicroProfile sample, else jump to [Steps](#steps)

[![Create Toolchain](https://github.com/IBM/container-journey-template/blob/master/images/button.png)](https://console.ng.bluemix.net/devops/setup/deploy/?repository=https://github.com/PierreFeillet/IBM-ODM-Kubernetes)

Please follow the [Toolchain instructions](https://github.com/IBM/container-journey-template/blob/master/Toolchain_Instructions.md) to complete your toolchain and pipeline.

## Steps

1. [Install Docker CLI and Bluemix Container registry Plugin](#1-install-docker-cli-and-bluemix-container-registry-plugin)
2. [Get and build the application code](#2-get-and-build-the-application-code)
3. [Build application containers](#3-build-application-containers)
4. [Create Services and Deployments](#4-create-services-and-deployments)

# 1. Install Docker CLI and Bluemix Container Registry Plugin


First, install [Docker CLI](https://www.docker.com/community-edition#/download).

Then, install the Bluemix container registry plugin.

```bash
bx plugin install container-registry -r bluemix
```

Once the plugin is installed you can log into the Bluemix Container Registry.

```bash
bx cr login
```

If this is the first time using the Bluemix Container Registry you must set a namespace which identifies your private Bluemix images registry. It can be between 4 and 30 characters.

```bash
bx cr namespace-add <namespace>
```

Verify that it works.

```bash
bx cr images
```


# 2. Get and build the application code

* Install the [Microservice Builder fabric](https://microservicebuilder.mybluemix.net/docs/installing_fabric_task.html) - which provides additional services that run on top of Kubernetes.

> **Note:** For the following steps, you can get the code and build the package by running the get_code.sh script present in scripts directory.

* `git clone` the following projects:
   * [odm-ondocker](https://github.com/lgrateau/odm-ondocker)
   ```bash
      git clone https://github.com/lgrateau/odm-ondocker
  ```

  ```
   * [Vote - for log](https://github.com/WASdev/sample.microservicebuilder.vote)
   ```bash
      git clone https://github.com/WASdev/sample.microservicebuilder.vote.git
      cd sample.microservicebuilder.vote/
      git checkout 4bd11a9bcdc7f445d7596141a034104938e08b22
  ```

* `mvn clean package` in each ../sample.microservicebuilder.* projects


# 3. Build application containers

Use the following commands to build the microservers containers.

Build the decision server container

```bash
docker tag dockercompose_decisionserverruntime:latest registry.ng.bluemix.net/<namespace>/dockercompose_decisionserverruntime:latest 
docker push registry.ng.bluemix.net/<namespace>/dockercompose_decisionserverruntime:latest  
```

Build the Derby decision db server container

```bash
dbserver
docker tag dockercompose_dbserver:latest registry.ng.bluemix.net/<namespace>/dockercompose_dbserver:latest  
docker push registry.ng.bluemix.net/<namespace>/dockercompose_dbserver:latest
```

Build the decision center container

```bash
dockercompose_decisioncenter
docker tag dockercompose_decisioncenter:latest registry.ng.bluemix.net/<namespace>/dockercompose_decisioncenter:latest  
docker push registry.ng.bluemix.net/<namespace>/dockercompose_decisioncenter:latest
```

Build the decision server console runtime container

```bash
dockercompose_decisionserverconsole
docker tag dockercompose_decisionserverconsole:latest registry.ng.bluemix.net/<namespace>/dockercompose_decisionserverconsole:latest  
docker push registry.ng.bluemix.net/<namespace>/dockercompose_decisionserverconsole:latest
```

Build the decision runner container

```bash
dockercompose_decisionrunner
docker tag dockercompose_decisionrunner:latest registry.ng.bluemix.net/<namespace>/dockercompose_decisionrunner:latest
docker push registry.ng.bluemix.net/<namespace>/dockercompose_decisionrunner:latest
```

```bash ToDo
cd sample.microservicebuilder.web-app
docker build -t registry.ng.bluemix.net/<namespace>/microservice-webapp .
docker push registry.ng.bluemix.net/<namespace>/microservice-webapp
```

# 4. Create Services and Deployments

Change the image name given in the respective deployment YAML files for  all the projects in the manifests directory with the newly build image names.

Get the public ip of the node

```bash
$ kubectl get nodes
NAME             STATUS    AGE
169.47.241.106   Ready     23h
```
Set the value of `SOURCE_IP` env variable present in deploy-nginx.yaml file present in manifests folder with the public ip of the node.

Deploy the microservice from the manifests directory with the command `kubectl create -f <filename>` or run the following commands:

```bash
kubectl run dbserver --image=registry.ng.bluemix.net/odmlab/dockercompose_dbserver:latest
kubectl expose deployment/dbserver --type=NodePort --port=1527 --name=dbserver

kubectl run decisionserverconsole --image=registry.ng.bluemix.net/odmlab/dockercompose_decisionserverconsole:latest
kubectl expose deployments decisionserverconsole --type=NodePort --port=9080 --name=decisionserverconsole
or kubectl expose deployment/decisionserverconsole --type=NodePort --port=9080,1883 --name=decisionserverconsole

kubectl run decisionserverruntime --image=registry.ng.bluemix.net/odmlab/dockercompose_decisionserverruntime:latest 
kubectl expose deployment/decisionserverruntime --type=NodePort --port=9080 --name=decisionserverruntime

kubectl run decisioncenter --image=registry.ng.bluemix.net/odmlab/dockercompose_decisioncenter:latest
kubectl expose deployments decisioncenter --type=NodePort --port=9060 --name=decisioncenter

kubectl run decisionrunner --image=registry.ng.bluemix.net/odmlab/dockercompose_decisionrunner:latest
kubectl expose deployments decisionrunner --type=NodePort --port=9070 --name=decisionrunner
```

After you have created all the services and deployments, wait for 10 to 15 minutes. You can check the status of your deployment on Kubernetes UI. Run 'kubectl proxy' and go to URL 'http://127.0.0.1:8001/ui' to check when the application containers are ready.

![Kubernetes Status Page](images/kube_ui.png)


After few minutes the following commands to get your public IP and NodePort number.

```bash
$ kubectl get nodes
NAME             STATUS    AGE
169.47.241.106   Ready     23h
$ kubectl get svc nginx-svc
NAME        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-svc   10.10.10.167   <nodes>       80:30056/TCP   11s
```

Now you can use the link **http://[IP]:30056** to access your application on browser.

Web application home page

![Web-app Home Page](images/ui1.png)

When you click on speaker name

![Speaker Info](images/ui2.png)

## Troubleshooting

* If your microservice instance is not running properly, you may check the logs using
	* `kubectl logs <your-pod-name>`
* To delete a microservice
	* `kubectl delete -f manifests/<microservice-yaml-file>`
* To delete everything
	* `kubectl delete -f manifests`


## References
* This java microservices example is based on Kubernete's [Microprofile Showcase Application](https://github.com/WASdev/sample.microservicebuilder.docs).

# License
[Apache 2.0](LICENSE)

