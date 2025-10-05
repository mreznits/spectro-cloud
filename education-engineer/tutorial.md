# Deploy an Application to a Local Kubernetes Cluster

[Kubernetes](https://kubernetes.io/) is an orchestration platform for deploying containerized applications. A Kubernetes cluster consists of a control plane plus a set of worker machines, called nodes. These nodes can be physical machines in a datacenter or virtual machines hosted on a cloud provider. When you deploy a program to a cluster, Kubernetes intelligently distributes work to the individual nodes for you.

Applications running on Kubernetes are packaged as containers. A container hosts the application code and all the dependencies that the app requires to run properly. Containers, in turn, are organized into pods, allowing them to share the same resources and local network. A single node can run one or more pods. 

[TODO: ... cloud environment, such as AWS or Azure?]: #
[TODO: Various sources describe the technology as Kind, KinD, or kind. Original tutorial used "kind" in the text, but "Kind" in the title. Which one should I use?]: # 
Although Kubernetes production clusters are typically hosted in a cloud environment, it is possible to create a Kubernetes cluster locally using the [Kubernetes-in-Docker (Kind)](https://kind.sigs.k8s.io/) command-line tool. Local deployment avoids the operational overhead of a full-blown cluster, allows for easy and efficient testing, and accelerates productivity.

This tutorial demonstrates how you can use Kind and Kubernetes to configure, deploy, and access a local application. 

# Prerequisites

Install Docker from https://docs.docker.com/install/ and install Kind from https://kind.sigs.k8s.io/.

Start Kind with the command `kind create cluster` and wait for the setup to complete.

```shell
$ kind create cluster
```

```shell
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

[TODO: Consider labelling this step as optional.]: #
To check for connectivity with the Kubernetes cluster and the Kubernetes API, use the following command:

```shell
$ kubectl cluster-info --context kind-kind
```

The command output contains the control plane IP:

```shell
Kubernetes control plane is running at https://127.0.0.1:51033
CoreDNS is running at https://127.0.0.1:51033/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

# Create Configuration File

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

The configuration file for the application *web* contains a *Deployment* configuration and a *Service* configuration. The *Deployment* configuration provides Kubernetes with the desired state of the application. The *Service* configuration exposes a port of the local cluster node (*NodePort*) to its external network, allowing you to access the application.

# Deploy Application

To deploy the application *web*, use the following command:

```shell
$ kubectl apply -f app.yaml
```

```shell
deployment.apps/web created
service/web created
```

# Access Application

To access the application, get the container name and use port forwarding to expose the container port to the local network.

[TODO: In Windows PowerShell, assigning the PODNAME variable requires $PODNAME=... I hesitate to update this without testing in Linux. Possibly two versions of the command might need to be provided here.]: #

To get the container name, use the following command:
```shell
$ PODNAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
```

The container name has the following format: `web-769bbccc48-v7ctx`.

To expose the port to the local network, use the following command:
```shell
$ kubectl port-forward $PODNAME 8080:8080
```

```shell
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

Visit localhost:8080 to see the application's Hello World welcome page:
```
Hello, world!
Version: 1.0.0
Hostname: web-769bbccc48-v7ctx
```

When you access localhost:8080, the command output prints the following:
```shell
Handling connection for 8080
```

# Cleanup

To delete the cluster, use the following command:
```shell
$ kind delete cluster
```

```shell
Deleting cluster "kind" ...
Deleted nodes: ["kind-control-plane"]
```

# Next Steps

This tutorial demonstrated how you can use Kind and Kubernetes to create a local cluster and deploy an application.

To learn how to use Spectro Cloud Palette to deploy a cluster to Amazon Web Services (AWS), Microsoft Azure, Google Cloud Platform (GCP), see [Palette Getting Started](https://docs.spectrocloud.com/tutorials/getting-started/palette/).
