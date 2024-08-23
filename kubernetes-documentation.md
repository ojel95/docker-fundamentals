# Kubernetes documentation

## What is Kubernetes?

Kubernetes is a collection of tools / services and concepts that help you deploy, scale and manage
containerized applications - typically across multiple hosts (i.e. multiple remote machines).

With Kubernetes, you can set up a configuration which will work on any host that supports
 Kubernetes - no matter if it's a cloud provider or your own, Kubernetes-configured data center.

- Kubernetes will not setup your infrastructure. You need to prepare the cluster, with all the
needed dependencies and services, where kubernetes will run. You can use tools and cloud provider that help
you with that like *Kubermatic* and *AWS EKS*

## Why Kubernetes?

- Kubernetes makes setting up and configuring complex deployments easy.
- Automatically monitor your containers and restart them if they go down.
- It makes scaling and load balancing easy (as it's built-in).
- Easy managing data in volumes across multiple machines.

## Key concepts notes

Check the [official resources documentation](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/) for more information.

### Cluster

Is a network of machines which are configured to support Kubernetes.
These machines are called **Nodes**

### Master node

Hosts the "Control Plane" - i.e. it's the control center which manages your
deployed resources.

The master node hosts a couple of tools/processes:

- **API Server**: Responsible for communicating with the Worker Nodes (e.g. to launch a new
Container)
- **Scheduler**: Responsible for managing the Containers, e.g. determine on which Node to
launch a new Container
- A couple of other things, also see the [official docs](https://kubernetes.io/docs/concepts/overview/components/)

### Worker nodes

May be a virtual or physical machine, depending on the cluster, where the actual Containers(Pods) are running on.

On Worker Nodes, we got the following running "tools" / processes:

- **kubelet service**: The counterpart for the Master Node API Server, communicates with the
Control Plane
- **Container runtime (e.g. Docker)**: Used for actually running and controlling the Containers
- **kube-proxy service**: Responsible for Container network (and Cluster) communication and
access

### Pod object

- Are the smallest unit in the Kubernetes world.
- Contains one or more Container (typically one) and any configuration as
well as volumes required by the Container(s).
- Has a cluster-internal IP by default
- Containers inside a Pod can communicate via localhost.
- Pods are designed to be ephemeral.

### Deployment object

- Controls (multiple) Pods.
- Used to set a desired state and Kubernetes then changes the actual state.
- Define which pods and containers to run and the number of instances.
- Deployments can be paused, deleted and rolled back.
- Can be scaled dynamically (and automatically)
- You can change the numbers of desired Pods as needed.
- Multiples deployments can be created.
- You don't control directly the pods, you use deployments to set up the desired end state.

#### Imperative approach

You can create a deployment using kubectl commands without a configuration file.
The image must be published.

```
kubectl create deployment first-app --image=orjoeslo/first-k8s-app
```

#### Declarative approach

When defining a Deployment config file you can declare one pod. For multiple pods you need multiple deployments.

### Service object

- Group Pods with a shared IP
- Can allow external access to Pods
- The default (internal only) can be overwritten
- Since Pods runs as the desired state of the deployment, every time a pod crash,
the deployment will restart the pod (The restart time gets longer every crash to prevent infinite loops)

One way instead of creating a service object is to expose the deployment
```
kubectl expose deployment first-app --type=LoadBalancer --port=8080
```

- The expose type can be:
  - ClusterIp: It will only be reachable from inside the cluster. Address still doesn't change
  - NodePort: The deployment should be exposed with the help of the IP of the worker node on which is running.
  It will be reachable from outside
  - LoadBalancer: Use the load balancer of the infrastructure. It will generate a unique addr for the service.
  Distributes all incoming traffic across all pods which are part of this service.
- The port is the app port that needs to be exposed.
- In the case of Rancher cluster I needed to add `--target-port=8080` in order to be available to access the node.

### Scaling

It refers to have multiple replicas (instances) of the same pod/container.

One way to add scaling to a deployment that doesn't have by configuration (for more or less replicas):
```
kubectl scale deployment/first-app --replicas=3
```

### Liveness Probe

This is an element of the Container in the Pod spec. It is used to monitor the health of the container.
In this example it checks the answer of the the Get request to the / path with port 8080 every 10 seconds
and delays 5 seconds before start checking the first time. For more documentation [Container resources](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#Container)

Check the file deployment.yaml to see where it's placed.

```
  livenessProbe:
    httpGet:
      path: /
      port: 8080
    periodSeconds: 10
    initialDelaySeconds: 5
```

### Volumes

- k8s can mount volumes into containers
  - Broad variety of Volume type/drivers are supported
    - local volumes
    - Cloud-provider specific Volumes
  - Volume lifetime depends on the Pod lifetime
    - Volumes survive Container restarts (and removal)
    - Volumes are removed when Pods are destroyed

## Commands

### Imperative deployment creation

You can create a deployment using kubectl commands without a configuration file.
The image must be published.

```
kubectl create deployment DEPLOYMENT-NAME --image=REPOSITORY/IMAGE-NAME
```

### Expose service

One way instead of creating a service object is to expose the deployment

```
kubectl expose deployment DEPLOYMENT-NAME --type=SERVICE_TYPE --port=PORT_NUMBER
```

- SERVICE_TYPE can be:
  - ClusterIp: It will only be reachable from inside the cluster. Address still doesn't change
  - NodePort: The deployment should be exposed with the help of the IP of the worker node on which is running.
  It will be reachable from outside
  - LoadBalancer: Use the load balancer of the infrastructure. It will generate a unique addr for the service.
  Distributes all incoming traffic across all pods which are part of this service.
- The port is the app port that needs to be exposed.

### Add Scaling

One way to add scaling to a deployment that doesn't have by configuration (for more or less replicas):

```
kubectl scale deployment/DEPLOYMENT-NAME --replicas=NUMBER_OF_REPLICAS
```

### Update deployment image

Update/change deployment image:

```
kubectl set image deployment/DEPLOYMENT-NAME CONTAINER_NAME=REPO/IMAGE_NAME:TAG
```

- The image name or tag needs to change in order to be updated.

### Rollout deployments

Check rollout status. This can be used when an new image is set in order to check if the update was successful.

```
kubectl rollout status deployment/first-app
```

Undo last deployment

```
kubectl rollout undo deployment/first-app
```

Check rollout deployment history

```
kubectl rollout history deployment/first-app
```

Get rollout revision details

```
kubectl rollout history deployment/first-app --revision=REVISION_NUMBER
```

Rollback to specific revision

```
kubectl rollout undo deployment/first-app --to-revision=3
```

### Apply configuration files

To create or update an object using a configuration file:

```
kubectl apply -f=FILE_NAME.yaml
```

### Delete objects

It can be imperative:

```
kubectl delete OBJECT OBJECT_NAME
```

It can be declarative pointing to the config file that create the object:

```
kubectl delete -f=FILE_NAME.yaml
```

It can be by label (The object should have a label as metadata). For security, so you don't delete unwanted objects,
you can specify the desired object kinds separated by comma:

```
kubectl delete deployments,services -l KEY=VALUE
i.e
 kubectl delete deployments,services -l group=example
```