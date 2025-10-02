[Original title ("deploy ... with Kind") is not meaningful to a user who is unfamiliar with Kind. What is an actual problem that this tutorial can help users solve?]: #

# Deploy an Application to a Local Kubernetes Cluster

[TODO: ... cloud environment, such as AWS or Azure?]: #
[Kubernetes](https://kubernetes.io/) is an orchestration platform for deploying containerized applications. Kubernetes production clusters are typically in a cloud environment. 

[TODO: Various sources describe the technology as Kind, KinD, or kind. Original tutorial used "kind" in the text, but "Kind" in the title. Which one should I use?]: # 
[Kubernetes-in-Docker (Kind)](https://kind.sigs.k8s.io/) is a command-line tool that enables developers to create a local Kubernetes cluster using docker images. Local deployment avoids the operational overhead of a full-blown cluster, allows for easy and efficient testing, and accelerates productivity.

This tutorial demonstrates how you can use Kind and Kubernetes to configure, deploy, and access a local application. 

# Prerequisites

[TODO: What is Docker?]: #
Install Docker from https://docs.docker.com/install/ and install Kind from https://kind.sigs.k8s.io/.

Start Kind with the command `kind create cluster` and wait for the setup to complete.

```
$ kind create cluster
```

```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Use the CLI to check for connectivity with the Kubernetes cluster and the Kubernetes API.

```
$ kubectl cluster-info --context kind-kind
```

[TODO: rewrite, esp "should" and "and more"]: #
You should see output that contains the control plane IP address and more. 

[TODO: insert output here]: #

## Create Configuration File

Create a file named **app.yaml** and insert the following configuration. 

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-app
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

["to its external Azure network"? Removed reference to Azure. AFAIK, we are not necessarily working with Azure.]: #

The configuration file for the application *web* contains a *Deployment* configuration and a *Service* configuration. The *Deployment* configuration provides Kubernetes with the desired state of the application. The *Service* configuration exposes a port of the local cluster node (*NodePort*) to its external network, allowing you to access the application.

## Deploy Application

[Note: this is local deployment (i.e. not to AWS / Azure)]: #

Deploy the application, *web*, using the following command.

```shell
$ kubectl apply -f app.yaml
```

## Access Application

To access the application, get the container name and use port forwarding to expose the container port to the local network.

To get the container name, issue the following command:

```shell
$ PODNAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
```

Now that you have the container name, expose the port to the local network.
```
$ kubectl port-forward $PODNAME 8080:8080
```

```
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

Visit localhost:8080 to see the Hello World welcome page.

# Cleanup

[TODO: are there any cleanup steps?]: #

# Next Steps

[TODO: Add actual next steps. Original text did not contain any next steps.]: #
