# Know how to use Ingress controllers and Ingress resources

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within a cluster. Ingress consists of two components. The `Ingress` Resource is a collection of rules for the inbound traffic to reach Services. These are Layer 7 (L7) rules that allow hostnames (and optionally paths) to be directed to specific Services in Kubernetes. The second component is the `Ingress Controller` which acts upon the rules set by the Ingress Resource, typically via an HTTP or L7 load balancer. It is vital that both pieces are properly configured to route traffic from an outside client to a Kubernetes Service.

Let's take the following example:

```mermaid
flowchart TD
    Client(External Client)

    subgraph Cluster [Kubernetes Cluster]
        IngressConfig(Ingress Resource: simple-fanout-example)
        
        IngressCtrl(Ingress Controller)

        subgraph RouteDefault [Default Route]
            direction LR
            SvcDefault(Service: default-service Port 80)
            PodDefault(Pod)
        end

        subgraph RouteFoo [Foo Route]
            direction LR
            Svc1(Service: service1 Port 4200)
            Pod1(Pod)
            
        end

        subgraph RouteBar [Bar Route]
            direction LR
            Svc2(Service: service2 Port 8080)
            Pod2(Pod)
        end
    end

    %% Configuration Relationship
    IngressConfig -.-> |Host: foo.bar.com| IngressCtrl

    %% Traffic Flow
    Client --> |Requests foo.bar.com| IngressCtrl
    
    IngressCtrl --> |Path: / | SvcDefault
    IngressCtrl --> |Path: /foo | Svc1
    IngressCtrl --> |Path: /bar | Svc2
    
    SvcDefault --> |Forwards traffic| PodDefault
    Svc1 --> |Forwards traffic| Pod1
    Svc2 --> |Forwards traffic| Pod2

    class Client client
    class IngressConfig config
    class IngressCtrl ctrl
    class SvcDefault,Svc1,Svc2 svc
    class PodDefault,Pod1,Pod2 pod
```

The corresponding yaml creates two ingress rules for the website foo.bar.com along with the default:

* The default path will direct traffic to the service “default-service” which listens on port 80
* Paths ending in /foo will direct traffic to the service “service1” which listens on port 4200
* Paths ending in /bar will direct traffic to the service “service2” which listens on port 8080

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-fanout-example
spec:
  rules:
    - host: foo.bar.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: default-service
                port:
                  number: 80
          - path: /foo
            pathType: Prefix
            backend:
              service:
                name: service1
                port:
                  number: 4200
          - path: /bar
            pathType: Prefix
            backend:
              service:
                name: service2
                port:
                  number: 8080
```

In order for the Ingress resource to work, the cluster must have an ingress controller running. Ingress controllers are deployed into the Kubernetes cluster as a workload:

```shell
> kubectl get po -A | grep nginx-ingress
ingress-nginx              nginx-ingress-controller-2gxtd                            1/1     Running     0          14d
ingress-nginx              nginx-ingress-controller-9lrzh                            1/1     Running     0          14d
ingress-nginx              nginx-ingress-controller-r2ksq                            1/1     Running     0          14d
```

The ingress controller itself is exposed as a `service` - Either `clusterIP`, `loadBalancer` or `nodePort` and acts as the entrypoint into the cluster for Ingress traffic.

!!! success "Exam Tip"

    Ingress is formed of two parts - the `controller` (in the datapah) and the `ingress` object itself (rules).

!!! success "Exam Tip"

    The ingress controller is exposed as a service, the service IP/FQDN is where we need to direct traffic to.