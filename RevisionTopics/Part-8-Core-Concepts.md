

**<span style="text-decoration:underline;">CKA Curriculum part 8 - Core Concepts</span>**

**<span style="text-decoration:underline;">Understand the Kubernetes API primitives</span>**

Quite a large subject to cover, but is well documented at [https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)

**<span style="text-decoration:underline;">Understand the Kubernetes cluster architecture</span>**



*   Master Nodes
    *   Etcd (unless external)
    *   Kube-APIserver
    *   Kube-Scheduler
    *   Kube-Controller-Manager
*   Worker Nodes
    *   Some sort of CNI (Flannel, NSX-T, etc)
    *   Kube-Proxy
    *   Kubelet
    *   Container runtime (Docker, RKT, containerd, etc)

![alt_text](https://i.imgur.com/wXKjYGD.png "image_tooltip")


**<span style="text-decoration:underline;">ETCD</span>**

A consistent and highly available key-value store leveraged by Kubernetes to store cluster related data. ETCD can be located on the master nodes, or can be a separate cluster all together.

**<span style="text-decoration:underline;">Kube-APIServer</span>**

Everything we do in Kubernetes goes through the API server. Think of this as as the front door to the Kubernetes cluster. This runs on all master nodes, to which a load balancer can be placed in front of.

**<span style="text-decoration:underline;">Kube-Scheduler</span>**

Authority for determining which pods run on which nodes. Kube-Scheduler will also respects any constraints or requirements imposed by the pod specification, such as any nodeselector configuration.

**<span style="text-decoration:underline;">Kube-Controller-Manager</span>**

Primarily responsible for checking, validating and rectifying current and intended state within the cluster. This includes:



*   Reacting to node failure.
*   Reacting to when the desired number of pods in a deployment (replication controller) is not satisfied.
*   Populating endpoints for services based on defined inclusion criteria (ie labels).

**<span style="text-decoration:underline;">CNI</span>**

Responsible for facilitating pod communication. A K8s cluster cannot function with a CNI.

**<span style="text-decoration:underline;">Kube-Proxy</span>**

Maintains network rules and provides a level of abstraction by forwarding connections as appropriate

**<span style="text-decoration:underline;">Kubelet</span>**

The kubelet takes a set of PodSpecs that are provided through various mechanisms (primarily through the apiserver) and ensures that the containers described in those PodSpecs are running and healthy.

**<span style="text-decoration:underline;">Container Runtime</span>**

Usually Docker or Containerd, this provides the platform for containers to run on the host.

**<span style="text-decoration:underline;">Understand Services and other network primitives</span>**

Services are categorised into the following types:

**ClusterIP** - Exposes the service on a IP address that is only accessible internally. Unless explicitly defined in the service yaml file, this is the default type.

**NodePort - **Exposes the service on each worker node’s IP address (non docker bridge) over a static port. This is accessible externally from the cluster.

**Load Balancer** - Exposes the service externally using a **cloud provider’s** load balancer. As implied, the cloud provider must support this functionality.
