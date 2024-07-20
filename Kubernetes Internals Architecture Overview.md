# Kubernetes Internals: Architecture Overview

With the brief introduction to the Kubernetes, it is important to make the next stop at understanding the internals of the K8s and getting used to all the intricate parts of the K8s system.

There are several key components involved, we are going to take a brief look at each of these components and understand the bigger picture of how all of them come together to work as one cool tool. Following the architecture, we also go over few very key concepts of the Kubernetes. Let’s crack on...

# **Kubernetes Cluster:**

K8s cluster is a collection of nodes where the containerised application will run. At a minimum, a cluster will have a **control plane or Master node** and one or more **computing nodes or Worker nodes.**

> If you are running K8s you are running a cluster.
> 

At a high-level master node controls the state of the cluster and the worker nodes run the application. For a cluster to be functional there should be at least one master and a worker node. A cluster might have more than one master node in a production setup and several worker nodes distributed across.

Kubernetes cluster as said can be split into two,

1. Master or Control plane
2. Worker Nodes

Each has its own separate internals shown below, let’s see each of these in detail and the role they play.

![https://miro.medium.com/v2/resize:fit:1400/1*LSIcBhCefJmosUuAXSEc_A.png](https://miro.medium.com/v2/resize:fit:1400/1*LSIcBhCefJmosUuAXSEc_A.png)

Internals of Control Plane and Worker Node

# **Control Plane:**

The control plane or Master node is the decision making part of the K8s cluster. The Control plane stores the desired state for the cluster from the declarative YAML file and keeps an eye out to make sure the current state of the cluster satisfies the desired state provided by the user. When the current state of the cluster is not desired state, the control plane pulls up everything to bring up the desired state in the cluster.

The Control plane has few key components to do all these magical things, let’s see their individual roles.

**etcd:**

etcd is the persistent store within the control plane. etcd is a key-value store where all the cluster data like the number of instances to run, how much the cluster scales up or down depending upon the network traffic, details about the namespace, internal DNS, etc. etcd is only accessible to the API server for security reasons.

> Any cluster related state or data will be stored to the etcd.
> 

**API server:**

K8s API server is the outward-facing component of the control plane. API server validates the API objects coming via the k8s manifest file or command line parameters like Pods, replica sets, services before configuring them into the cluster. K8s API server is also responsible for the communication between the control plane and the worker nodes. The API server is usually accessed from the outside world via CLI tools like “Kubectl”.

> API server is the front end of the control plane, handling all the internal and external requests.
> 

**Control manager:**

The control manager or controller is the watch-hawk of the control plane. It monitors the cluster continuously to make sure the current state of the cluster is the same as the desired state. Whenever the current state drifts from the desired state control manager inform the API server to do the necessary actions. There are several different types of controllers available each Kubernetes resource type.,

1. ReplicaSet
2. Deployment
3. StatefulSet
4. Job
5. CronJob
6. DeamonSet

It is also worth noting that controllers can be run outside the control plane and it is also possible to write custom controllers to satisfy distinct needs.

> Controllers are control loops that watch the state of your cluster and tries to move the current cluster state closer to the desired state.
> 

**Scheduler:**

Whenever the control plane needs to spin a new Pod it is the responsibility of the scheduler to find a feasible node or available node to bind that in. It scans all the nodes for availability and binds that to the new or unscheduled pod. If none of the nodes is available the Pod remains unscheduled until a node becomes available. The Scheduler is also responsible for monitoring the usage of the hardware resources such as CPU, memory to be aligned with the usage policies and the overall health of the cluster.

Similar to controllers it is possible to do a custom implementation of schedulers as well.

> Scheduler becomes responsible for finding the best Node for that Pod to run on.
> 

# **Worker Node:**

The worker node is responsible for running the application and workloads. Each worker node is controlled by the control plane. In a production setup, there will be several worker nodes. Worker nodes can be physical machines or VMs depending upon the cluster configuration.

let see the internal of the worker node brief.

**Kubelet:**

Kubelet takes instruction from the control plane and makes sure the Pod bonded with the node is running healthy. This also reports to the control plane about the pod health and status.

**Kube-proxy:**

Kube-proxy is ****a proxy service that runs on each worker node to deal with isolated host subnetting and expose services to the outside world. It is also responsible to programs the “iptables” rules on the node for forwarding the request to the correct pods across the various isolated networks in a cluster.

**Container runtime:**

The engine that runs takes the containerised application and run them on the node. Each node needs to have a container runtime to be functional. Below are a few popular options,

- [CRI-O](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)
- [containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
- [Docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

# **How Does It All Work Together?**

Now that we have seen all of the individual components, let's glimpse how all of these come together to run a container cluster.

Kubernetes follows the **declarative** model, what that means is as the user you provide K8s what are you need in the cluster and K8s does all the heavy lifting to maintain the desired state provided by the user. Those declarative commands user provides are referred to as **manifest,** often a YAML file with predefined keys and values.

Once the manifest file is made available to K8s, the master node kicks in. The scheduler in the master node after receiving the manifest file sends those instructions down to the controller. The controller now tries to find an available worker node or create a new one and send the necessary information to the Kubelet. Kubelet takes the instruction and spins up the new application instance inside the worker node. Once the new pod is up and healthy it's now the responsibility of the kubelet to send the health status of the pod to the controller. If something goes wrong like the pod shutdown or pod failure controller right away spins up a new one in any available worker node by sending out the necessary commands to the kubelt thereby keeping the desired number of pods match the currently running instances.

## **Conclusion:**

This article on the internals of the Kubernetes is barely scratching the surface of it. There are a whole lot of other concepts that are hidden under the hood. But this article provides a good foundation and a bird’s eye view on Kubernetes. From here we are going to dig deeper into the key components within the Kubernetes. Next up Pods in Kubernetes.