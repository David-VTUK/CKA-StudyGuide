# Use ClusterIP, NodePort, LoadBalancer service types and endpoints

Pods are ephemeral. Therefore, placing these behind a service which provides a stable, static entrypoint is a fundamental use of the kubernetes service object. To reiterate, services take the form of the following:

* `ClusterIP` - Internal only
* `LoadBalancer` - External, requires cloud provider, or software implementation to provide one
* `NodePort` - External, requires access the nodes directly

The 
