# Why do you need a Service Mesh as well as a CNI?

## When you set up your Kubernetes cluster, you may have installed a Container Network Interface (CNI) component such as Calico or Flannel. You may be told you need to install a Service Mesh, such as Istio. In this article we look at why you may want to.

This article is not meant to provide a comprehensive description of Kubernetes networking but introduces enough of it to help you understand the difference between a CNI supplier, such as Calico, and a Service Mesh supplier, such as Istio.

![https://miro.medium.com/v2/resize:fit:1400/1*y1ZFdGJmA7CzIhFa3uvOxw.png](https://miro.medium.com/v2/resize:fit:1400/1*y1ZFdGJmA7CzIhFa3uvOxw.png)

Istio Service Mesh

# **Kubernetes Networks**

It is first important to understand that there are four networks in a Kubernetes cluster:

![https://miro.medium.com/v2/resize:fit:1400/1*tDsSyEtZBnZmhBD070beiQ.png](https://miro.medium.com/v2/resize:fit:1400/1*tDsSyEtZBnZmhBD070beiQ.png)

Kubernetes network layers

## **1 — Host network**

This is the network that connects all the hosts that are running as Kubernetes Nodes. IP addresses allocated within this network are defined within your Virtual Private Cloud (VPC).

## **2 — Cluster network**

This is the network that seamlessly connects all the Pods together. Kubernetes requires that all Pods can talk to one another without any Network Address Transaction (NAT).

Effectively this makes Pods work like they are virtual machines sitting on a virtual network. The virtual network is a subnet shared amongst all Pods.

If you have access to a Kubernetes cluster and `kubectl`, you can find the definition of this subnet with:

```
kubectl cluster-info dump | grep -m 1 cluster-cidr
```

It will show the CIDR of the cluster network, eg:

```
 "--cluster-cidr=192.168.0.0/16",
```

## **3 — Service Network**

Pods are ephemeral. This means that, at any moment, A Pod may be terminated and be rescheduled on the same or different Node. Even if it is on the same Node, it may be given a different IP address. Additional Pods may also be scheduled.

Because of this, Kubernetes introduces the idea of a Service. A Service is persistent and allows a client to access the Service even though the underlying Pods that provide the Service may change.

For this reason, each Service is given an IP address from the Service Network when it is created.

You can find the definition of this network with:

```
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
```

This will give you a result like:

```
"--service-cluster-ip-range=10.96.0.0/12",
```

Note that it is different to the cluster network. They must not overlap. You can define the ranges when you create the Kubernetes cluster but cannot change them afterwards.

## **4 — Container network**

Whilst you will normally only find a single container within a Pod, you may actually find multiple containers in the Pod, especially when using sidecars.

> A sidecar is an application that resides in its own container and provides functionality around the main application.
> 

When you have multiple containers in a Pod, they share the same network. They can be thought of as processes running in a virtual machine, which is the Pod. This means they all have access to the same, Pod IP address.

Like all processes running in a host, they can talk to each other via the `localhost` loopback network. As they share the same IP address, they must operate on different ports.

> Allocating ports within a Kubernetes cluster needs to be done carefully as they may need to be defined within the container, Pod, Service and Node networks.
> 

# **Container Network Interface (CNI)**

As mentioned above, Kubernetes requires that all of its Pods can talk to each other over the Cluster network. This requires each Node that is added to the cluster and each Pod that is scheduled, must be configured to implement the cluster subnet.

This is where the Container Network Interface (CNI) comes in.

When you install Kubernetes on a Node, you install a small application called `kubelet`. This acts as an Kubernetes agent on the Node, doing whatever it is told by the Kubernetes controller, including scheduling Pods.

When `kubelet` schedules a Pod, it need to set up the networking including it’s IP address. It does this by calling the CNI, which then assigns an IP address and configures the network interfaces within the Pod to ensure there is the connectivity that Kubernetes requires.

The CNI is an interface. The implementation of the interface is provided as a ‘plugin’ to Kubernetes. You will see this called a CNI plugin.

There are a number of CNI plugins, including Flannel and Calico. They implement the CNI in different ways within the Pod. They also provide different layers of security within the cluster.

> CNI is a mandatory requirement for Kubernetes.
> 

# **Service Mesh**

A Service Mesh is a layer that sits on the Cluster Network. Instead of providing ***open access*** to any Pod from any Pod, it is designed to secure the communications between specific Pods.

A Service Mesh operates as a sidecar to each main application container. As such, it provides network connectivity to your main application container as a network proxy. The connection between the proxy and the main application is by way of the `localhost` network.

![https://miro.medium.com/v2/resize:fit:1400/1*uwK0yibwA-YEEoonMhu3eg.png](https://miro.medium.com/v2/resize:fit:1400/1*uwK0yibwA-YEEoonMhu3eg.png)

Service Mesh

You can configure the proxy using Kubernetes manifest files. These are picked up by a control plan component which then translates the manifest files and sends the results to the proxy. The proxy then implements the rules that have been set up.

# **Do I need a Service Mesh and a CNI Plugin?**

In short, yes.

## **Need for CNI plugin**

Kubernetes requires that you implement the CNI interface so that it has connectivity between the Pods over the cluster network. It is a mandatory requirement.

The cluster network is completely open, by design. Any Pod can communicate to any other Pod.

In addition, the cluster network does not employ any encryption. If you need encryption, you can set it up between Pods but then you must distribute the keys and certificates for the TLS authentication and encryption. This can be very tedious and error prone.

## **Need for Service Mesh**

The Service Mesh runs on top of the cluster network and is optional.

The benefits of running a Service Mesh are:

1. Security … the Service Mesh can implement routing rules as well as mutual Transport Layer Security (mTLS) between Pods
2. Load balancing … define the distribution of traffic between Pods
3. Monitoring … provide metrics and logs on connections and traffic
4. Separation of concerns … allows network management to be abstracted from the application, making implementation consistent and technology agnostic for the application, including separation by namespace
5. Reliability … requests that fail can be retried automatically

All these benefits make for a convincing reason for implementing a Service Mesh on top of the CNI plugin.

## **Need for an API Gateway**

I have been asked on a number of occasions whether the additional cost or complexity of an API Gateway is necessary if you have installed a Service Mesh.

Whilst a Service Mesh can provide a number of API Gateway features, it is not designed for this specific use. As such, a Service Mesh will fall short of being a replacement for the API Gateway.

Some claim that your API Gateway is for managing North/South traffic (ie: to and from an external client), whereas Service Mesh manages East/West traffic (ie: between Pods). Whilst this makes it easier to understand, it is not, necessarily the case.

Your API Gateway can provide:

- Routing (including load balancing
- Authentication and authorisation
- Circuit breaker
- Service discovery
- Ingress management

## **Overlaps in solution**

Hopefully you can see that the CNI, Service Mesh and API Gateway all overlap. The trick is to use the component that best fits your needs (noting that the CNI is mandatory).

![https://miro.medium.com/v2/resize:fit:1400/1*YVls-huHKU_LCuitFSuyMg.png](https://miro.medium.com/v2/resize:fit:1400/1*YVls-huHKU_LCuitFSuyMg.png)

Service Mesh vs API Gateway vs CNI

In reality, there is no harm in having all three but costs may rise with the increased complexity and maintenance of the solution.

## **Further confusion**

If things were getting clearer, just be aware that this space is evolving and is becoming quite competitive.

For instance, Istio, which is a Service Mesh, now offers Istio CNI as a CNI plugin. Calico, which is a CNI plug in offers Pod to Pod security policies.

In place of the Service Mesh proxies, there are now eBPF solutions. eBPF (originally but no longer standing for extended Berkley Packet Filter) is a way to securely and reliably add scripted solutions to the operating system without the need to recompile the kernel.

eBPF now allows proxy and IP Tables solutions to be replaced with something far more efficient.

Several technologies are now producing eBPF based solutions.

## **Options for Service Mesh**

There are several technologies for the Service Mesh, including:

- Istio
- Linkerd
- Cilium
- Consul Connect
- Cloud specific solutions, eg: GCP Anthos, AWS App Mesh, Azure Open Service Mesh (deprecated)

Many of these are now charging for the service. Istio is still a free open-source solution but can be complicated to set up and debug.

# **Summary**

In this articles we have looked at:

- The word of Kubernetes networking
- What the Container Network Interface (CNI) is
- What a Service Mesh is
- Why we need a CNI even if we have a Service Mesh
- The need for an API Gateway
- The overlaps in networking solutions
- A list of available Service Mesh technologies

I will be adding an article on Istio installation and configuration.

![https://miro.medium.com/v2/resize:fill:88:88/1*dmbNkD5D-u45r44go_cf0g.png](https://miro.medium.com/v2/resize:fill:88:88/1*dmbNkD5D-u45r44go_cf0g.png)