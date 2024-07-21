# Kubernetes Hardening

The [adoption of Kubernetes](https://www.dynatrace.com/news/blog/kubernetes-in-the-wild-2023/) has demonstrated impressive growth over recent years, [with 96% of organisations](https://www.splunk.com/en_us/blog/learn/state-of-kubernetes.html) using or evaluating its use. As Kubernetes continues to evolve as the go-to container orchestrator, an understanding of its security implications becomes crucial.

This article explores the 2022 [Kubernetes Hardening Guide](http://summarize/) published by the NSA and CISA.

## **Pod Security.**

*Pods* are the fundamental component of Kubernetes, consisting of containers, and are generally the first step in an attacker’s initial access to the Kubernetes cluster. Aside from improving the security of the hosted application, a number of steps should be taken to improve the security of pods more generally.

**Image Security**. Generally, container images are built on top of existing ones. As a result, a whole variety of software may be available on the container — each with their own potential vulnerabilities. Instead, pod images built [from scratch](https://hub.docker.com/_/scratch) which only provide *exactly* the software required minimises the risk of initial- and post-exploitation.

On top of this, *image scanners* play an important role in checking image software for vulnerabilities. These can be implemented with an [*admission controller*](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook), such as [Trivy-Enforcer](https://github.com/aquasecurity/trivy-enforcer), to verify the images of all pods created. They may be *mutating*, where the pod specification (or Kubernetes object more generally) is modified before creation, or *validating* where creation requests are simply denied or accepted.

![https://miro.medium.com/v2/resize:fit:1116/1*A087fuzGfR_sZ0DSDhd4oA.png](https://miro.medium.com/v2/resize:fit:1116/1*A087fuzGfR_sZ0DSDhd4oA.png)

Validating Admission Controller for Image Scanning

**Security Context**. A pod’s [*security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)* can control a number of security features such as read-only file systems, runtime privileges, controlled privilege escalation, SELinux, Capabilities, seccomp and AppArmor.

**Seccomp & App Armor.** Linux kernel modules such as [Seccomp](https://en.wikipedia.org/wiki/Seccomp) and [AppArmor](https://apparmor.net/) enable [Mandatory Access Control](https://www.linkedin.com/pulse/overview-dac-mac-access-control-definitions-strengths-jay-/) (MAC), as opposed to the default Discretionary Access Control (DAC) of Linux. These can enable fine-grained control over system calls and file access.

In the context of Kubernetes, Seccomp profiles [can be used](https://kubernetes.io/docs/tutorials/security/seccomp/) to control container access to the underlying host by restricting (or logging) calls to `reboot`, `ioperm`, or `mount` for example. AppArmor can be used to [prevent writes](https://kubernetes.io/docs/tutorials/security/apparmor/#example) to sensitive files within the container.

**Immutable File Systems.** To limit post-exploitation activity, the root file systems of applicable pods should be set to read-only. This mitigates the risk of an attacker being able to modify the running applications and download tools or malware.

In the case applications require write access to the filesystem, `emptyDir` volumes can be mounted to specific directories — the `medium` of which can additionally be set to `Memory`.

Consider the following pod specification, defining a security context with a read-only file system, both Seccomp and AppArmour profiles and specific runtime user and Linux capabilities.

```
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
spec:
  containers:
  - name: my-app
   image: my-app:latest
   volumeMounts:
    - name: log
     mountPath: /var/log/my-app
   securityContext:
    runAsUser: 1000
    readOnlyRootFilesystem: true
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/fine-grained.json
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
  volumes:
   - name: log
    emptyDir:
     medium: "Memory"
```

**Enforcing Pod Security.** Prior to being deprecated, [Pod Security Policies](https://kubernetes.io/docs/concepts/security/pod-security-policy/) (PSPs), implemented through a built-in admission controller, could be used to enforce the creation of pods according to specified requirements. Instead, Kubernetes now provides an admission controller known as [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/) (PSA).

PSA defines requirements for a pod’s security context based on the defined [*Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/).* Currently, three standards exist (either `privileged`, `baseline` or `restricted`) and each define constraints on security contexts and pod specification. For example, the `restricted` standard only allows the `NET_BIND_SERVICE` capability to be added to a container.

Given these, PSA can be configured to log, warn or prevent the creation of pods (within a namespace) that don’t meet a specified standard. Enforcing more granular or custom access control requirements is left up to third-party solutions such as [OPA Gatekeeper](https://www.openpolicyagent.org/docs/latest/kubernetes-introduction/).

**Service Accounts.** [*Service accounts*](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) are used to provide access to the Kubernetes API for applications running within pods. By default, the credentials of these accounts are automatically mounted into the pod at known locations. It is important to ensure the authorization of these accounts apply the principle of least privilege, and the credentials are unmounted when not required through the `automountServiceAccountToken` parameter.

**Secrets**. By default, Kubernetes [secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/) are stored unencrypted as base64-encoded strings. Whilst access can be controlled through RBAC, secrets should additionally be [encrypted at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#ensure-all-secrets-are-encrypted) by using the API server’s `--encryption-provider-config` parameter.

## **Network Security.**

**Network Policies**. As in traditional environments, proper network segmentation is critical. Most obviously [*Network Policies*](https://kubernetes.io/docs/concepts/services-networking/network-policies/) are used for this (assuming the underlying [CNI](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) supports it). By default, the [Kubernetes network model](https://kubernetes.io/docs/concepts/services-networking/) does not prescribe a zero-trust network, with traffic being allow-by-default. Instead, a deny-by-default policy is important to implement using the below example. Note that network policies are additive.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Worker Node Segmentation.** Worker nodes host the user pods of the cluster. Proper segmentation between these nodes limits the risk of lateral movement and post-exploitation activity.

**CNI.** Whilst Kubernetes defines networking requirements, the implementations of these are left up to the [*Container Network Interface](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)* (CNI) *plugin* — an exchangeable component chosen based on cluster requirements. Different CNIs provide varying support for security-related features such as network policies, encryption and network monitoring.

**Service Mesh.** Built on top of the CNI, a [*service mesh*](https://www.weave.works/blog/introduction-to-service-meshes-on-kubernetes-and-progressive-delivery#:~:text=A%20Kubernetes%20service%20mesh%20is,and%20builds%20on%20its%20capabilities.) implements a network infrastructure with its own control and data planes through which traffic is centrally managed and routed. Implementing a service mesh can provide better observability through metrics and monitoring and security through encryption, authentication and authorization policies.

![https://miro.medium.com/v2/resize:fit:1400/1*aTIZQNzTgupcwgpLLu16dg.png](https://miro.medium.com/v2/resize:fit:1400/1*aTIZQNzTgupcwgpLLu16dg.png)

Example Service Mesh

**Resource Quotas**. Kubernetes provides support for [resource quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) to avoid CPU and memory exhaustion. These can be scoped to particular namespaces, nodes or pods. Similarly, [PID limits](https://kubernetes.io/docs/concepts/policy/pid-limiting/) and reservations can be enforced to mitigate control plane components from being starved.

## **Control Plane Hardening.**

The [*control plane*](https://kubernetes.io/docs/concepts/overview/components/) hosts the components responsible for managing the cluster — an ideal [target for attackers](https://cloud.hacktricks.xyz/pentesting-cloud/kubernetes-security/attacking-kubernetes-from-inside-a-pod). Control plane components such as the *kube-scheduler*, *etcd* and others can be implemented as pods found in the *kube-system* namespace. As such, network policies only allowing the minimum access needed to this namespace are important.

![https://miro.medium.com/v2/resize:fit:1400/0*IeLcCyMh_2GlfyKp.png](https://miro.medium.com/v2/resize:fit:1400/0*IeLcCyMh_2GlfyKp.png)

Kubernetes Control Plane.

**Kube-apiserver.** Specific components of the control plane require additional considerations too. For the [*API server*](https://kubernetes.io/docs/concepts/overview/kubernetes-api/), which provides management access to resources within the cluster, this includes [enabling TLS](https://kubernetes.io/docs/concepts/security/controlling-access/) and disabling anonymous access and its [insecure port](https://www.stigviewer.com/stig/kubernetes/2021-04-14/finding/V-242386#:~:text=By%20default%2C%20the%20API%20server,bypass%20authentication%20and%20authorization%20checks.) (if it exists).

Access to the API server is often authenticated by credentials and information found within [*kubeconfig*](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) files. These should be treated carefully, with appropriate read / write permissions and other controls in place relevant to sensitive credentials.

**Etcd.** The [*etcd component*](https://kubernetes.io/docs/concepts/overview/components/#etcd) stores cluster data and state information — with (write) access to this an attacker potentially has root access to the entire cluster. To mitigate this risk, only the API server should have access to etcd. Commonly, both components are hosted on the same node, but hosting them separately enables the possibility of implementing better firewalling. Additionally, setting up dedicated PKI infrastructure for etcd (and the API server) can better segment the two components from the rest of the cluster.

**Kubelet.** Whilst many control plane components (such as those considered above) are commonly hosted on dedicated control plane nodes, an instance of the [*kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)* runs on all worker nodes. This communicates with the API and is responsible for managing pods for the node. As such, having compromised a node, attackers can [target the kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) to [interface](https://github.com/cyberark/kubeletctl) with the control plane.

In default environments, access to the kubelet is unrestricted. Instead, anonymous access should be disabled by setting `--anonymous-auth` to false and proper authentication and authorization should be configured in the kubelet [config](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/).

## **Authentication & Authorization.**

**Authentication.** Aside from *service accounts* for pods, Kubernetes does not implement [authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) [solutions](https://kubernetes.io/docs/concepts/security/hardening-guide/authentication-mechanisms/) for users and admins in the traditional sense. Instead, it assumes a solution independent of the cluster exists. Most easily X.509 certificates can be used. In [this](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user), a client presents a certificate signed by the cluster’s CA and the common name is taken as the username.

Other authentication mechanisms use [bearer tokens](https://swagger.io/docs/specification/authentication/bearer-authentication/) where the API server can check validity in a number of ways. The simplest approach simply uses a [static file](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file) to store valid tokens, but support for [OAuth2 identity providers](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens) and [webhook authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication) exists.

Aside from X.509 certificates and bearer tokens, an [authenticating proxy](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authenticating-proxy) can be configured. In this case, the API server is configured to identify users based on provided HTTP headers which would be set by the proxy. To prevent spoofing a valid certificate is required on each request.

Support for anonymous access to the API should be disabled by passing the `--anonymous-auth=false` flag to the API server.

**Authorization.** Kubernetes supports [role-based access control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (RBAC) implemented through the `Role(Binding)` API objects. `Roles` define a set of allowed actions on objects and are assigned to particular usernames, groups or service accounts by creating a `RoleBinding`.

The above objects define permissions scoped to particular namespaces. However, the `ClusterRole` and `ClusterRoleBinding` objects can provide cluster wide access. Privileges assigned to users should apply the principal of least privilege.

**On-Premise vs Cloud**. Many clusters are hosted through cloud providers. In these cases the authorization and authentication solutions vary and are often integrated with the provider’s own IAM solutions such as [AWS](https://marcincuber.medium.com/amazon-eks-rbac-and-iam-access-f124f1164de7), [GCP](https://engineering.sada.com/gke-authentication-and-authorization-between-cloud-iam-and-rbac-d54ea17721d6) or [Azure](https://medium.com/microsoftazure/azure-kubernetes-service-aks-authentication-and-authorization-between-azure-rbac-and-k8s-rbac-eab57ab8345d).

## **Logging.**

Capturing activity within an environment is critical for auditing, investigations and monitoring. In Kubernetes there are many layers at which logging should be configured.

**Audit Logging.** All [API activity](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/) within a cluster, such as the creation of resources, occurs via the *kube-apiserver.* An *audit policy,* passed through the `--audit-policy-file` parameter, specifies [*audit rules](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)* to define which events to record and at what detail level. Events can be [recorded](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#audit-backends) to a local file or to a remote web API using a webhook.

```
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived" # Don't generate events for RequestReceived stage.
rules:
  # Record the request and response of events regarding secrets.
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["secrets"]
  # Record the metadata (user, timestamp, resource, verb, etc.) of calls to pod logs and pod status.
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]
```

**Container Logging.** By default Kubernetes logs the output of each container running within a pod. The *kubelet* on each worker is responsible for managing these, outputting them to`/var/log/{pods, containers}` and rotating them. Since these are local to each node, a proper solution aggregating these should be implemented.

Existing solutions such as [fluentd](https://docs.fluentd.org/v/0.12/articles/kubernetes-fluentd) can streamline this process, by implementing a *DaemonSet* on each node responsible for managing the logs. The NSA’s hardening guide recommends other similar solutions to log aggregation, such as using a [*sidecar container*](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) for each pod.

**Metrics.** Kubernetes ****implements a basic [metric pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/) through the [metrics-server](https://github.com/kubernetes-sigs/metrics-server), exposing basic CPU and memory usage information through the API for auto-scalers. However, the most common, more [complete solution](https://medium.com/@sushantkapare1717/setup-prometheus-monitoring-on-kubernetes-using-grafana-fe09cb3656f7), uses a combination of Prometheus for collection and Grafana for visualisation.

**Operating System Calls.** As previously discussed, Kubernetes support for *seccomp* can be used to limit a container’s access to the underlying host. However, its [audit mode](https://itnext.io/seccomp-in-kubernetes-part-i-7-things-you-should-know-before-you-even-start-97502ad6b6d6) can additionally be used for logging syscall activity.

# **Threat Detection.**

Successful defence not only requires the logging solutions previously discussed, but also active analysis of the collected data for potential threats. To do so, defenders require an understanding of [techniques used by attackers](https://cloud.hacktricks.xyz/pentesting-cloud/kubernetes-security) when infiltrating a cluster.