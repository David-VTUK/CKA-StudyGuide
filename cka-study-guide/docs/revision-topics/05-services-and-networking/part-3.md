# Use ClusterIP, NodePort, LoadBalancer service types and endpoints

Pods are ephemeral. Therefore, placing these behind a service which provides a stable, static entrypoint is a fundamental use of the kubernetes service object. They can be one of the following types:

* `ClusterIP` - Internal only
* `NodePort` - External, requires access the nodes directly
* `LoadBalancer` - External, requires cloud provider, or software implementation to provide one

We can think of these services like loadbalancers, with automatic endpoint management based on label selectors.

## Cluster IP

This is the default service type which exposes a set of Pods via a cluster-internal IP. It cannot be reached from outside of the Kubernetes Cluster

```mermaid
flowchart TD
    ExtClient(External Client - Unreachable)
    
    subgraph Cluster [Kubernetes Cluster]
        PodClient(Pod inside Cluster)
        Service(Service: ClusterIP 10.96.0.1)
        Target1(Pod: Target App)
        Target2(Pod: Target App)
    end

    ExtClient -.-> |Cannot connect| Service
    PodClient --> |Requests| Service
    
    Service --> Target1
    Service --> Target2

    classDef blocked fill:#1E2D3D,stroke:#F44336,stroke-width:2px,color:#fff

    class PodClient client
    class ExtClient blocked
    class Service svc
    class Target1,Target2 pod
```

The yaml for which looks like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-clusterip
  namespace: myapp
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

We define the service type in `spec.type`

We definite which Pods we want to target in `spec.selector`, where we specify the key:value pair of labels. For this specific example, it would look like:

```mermaid
flowchart LR
    subgraph Cluster [Kubernetes Cluster]
        Client(Internal Cluster Client)
        
        subgraph Namespace [Namespace: myapp]
            direction TD
            Service(Service: backend-clusterip Port 80)
            Pod1(Pod: app=backend Port 8080)
            Pod2(Pod: app=backend Port 8080)
        end
    end
    
    Client --> |Accesses Port 80| Service
    Service --> |Routes to TargetPort 8080| Pod1
    Service --> |Routes to TargetPort 8080| Pod2
        
    class Client client
    class Service svc
    class Pod1,Pod2 pod
```

## NodePort

A `nodePort` service works by exposing a high-value port directly on worker nodes. We can then direct traffic to one of these nodes on the specified port to access our application, even from outside of the cluster.

```mermaid
flowchart TD
    ExtClient(External Client)

    subgraph Cluster [Kubernetes Cluster]
        PortA(Node A - NodePort 30000)
        PortB(Node B - NodePort 30000)

        subgraph Namespace [Namespace: myapp]
            direction LR
            Service(Service: backend-nodeport Port 80)
            Pod1(Pod: app=backend Port 8080)
            Pod2(Pod: app=backend Port 8080)
        end
    end

    ExtClient --> |Accesses NodeIP:30000| PortA
    ExtClient --> |Accesses NodeIP:30000| PortB
    
    PortA --> |Forwards to Port 80| Service
    PortB --> |Forwards to Port 80| Service
    
    Service --> |Routes to TargetPort 8080| Pod1
    Service --> |Routes to TargetPort 8080| Pod2

    class ExtClient client
    class PortA,PortB node
    class Service svc
    class Pod1,Pod2 pod
```

The corresponding yaml looks like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-nodeport
  namespace: myapp
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30000
```

## Loadbalancer

A `loadBalancer` service type exposes the service on a dedicated VIP, usually provided by a cloud provider or software-based solution (metallb, Cilium, etc).

Note we can consider the `loadBalancer` service type a wrapper around `NodePort`.

```mermaid
flowchart TD
    ExtClient(External Client)
    ExtLB(External Load Balancer - Public IP)

    subgraph Cluster [Kubernetes Cluster]
        NodeA(Worker Node A - Auto NodePort)
        NodeB(Worker Node B - Auto NodePort)

        subgraph Namespace [Namespace: myapp]
            direction LR
            Service(Service: backend-loadbalancer Port 80)
            Pod1(Pod: app=backend Port 8080)
            Pod2(Pod: app=backend Port 8080)
        end
    end

    ExtClient --> |Accesses Public IP| ExtLB
    
    ExtLB --> |Routes to Nodes| NodeA
    ExtLB --> |Routes to Nodes| NodeB
    
    NodeA --> |Forwards to Port 80| Service
    NodeB --> |Forwards to Port 80| Service
    
    Service --> |Routes to TargetPort 8080| Pod1
    Service --> |Routes to TargetPort 8080| Pod2

    class ExtClient client
    class ExtLB lb
    class NodeA,NodeB node
    class Service svc
    class Pod1,Pod2 pod
```

The yaml for which looks like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-loadbalancer
  namespace: myapp
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

## Endpoints

Endpoints are essentially the destination points for a service traffic, regardless of type. They also provide a useful troubleshooting tool to determine if there are `Pods` active for a given service. For example:

```yaml
david@fedora:~/cka$ kubectl get endpoints argocd-redis -n argocd
# Note the Endpoints IP
NAME           ENDPOINTS       AGE
argocd-redis   10.0.1.4:6379   40d

# Note the IP address of the Pod
david@fedora:~/cka$ kubectl get po -n argocd -o wide | grep -i redis
argocd-redis-7f9487d4fd-knhpn                       1/1     Running   2 (8d ago)      24d   10.0.1.4     srv-rk1-01   <none>           <none>
```

We know the `argocd-redis` service has a single active endpoint.

!!! success "Exam Tip"

    Just because a service object has an IP address doesn't mean it will forward traffic to a working Pod.

!!! success "Exam Tip"

    If you're not getting traffic back from a `service` object, check its `endpoints`.

!!! success "Exam Tip"

    If a `service` object has endpoints, but is not returning any traffic, check which port it's forwarding traffic to on the `pod`, and which ports the `pod` is configured to listen on.