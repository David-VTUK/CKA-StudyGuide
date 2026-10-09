# Understand and use CoreDNS

From many versions ago, coredns has replaced kube-dns as the facilitator of cluster DNS and runs as pods.

```shell
kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-hxbhn                 1/1     Running   9          10d
coredns-fb8b8dccf-jks6g                 1/1     Running   4          8d
```

To view the DNS configuration of a pod, spin one up and inspect the /etc/resolv.conf:

```shell
kubectl run busybox --image=busybox -- sleep 9000
kubectl exec -it busybox sh
/ # cat /etc/resolv.conf  
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local virtualthoughts.co.uk
options ndots:5
```

“10.96.0.10” references the `coredns` service

“default.svc.cluster.local” References the namespace with the suffix svc.cluster.local.

All pods are provisioned a DNS record and are in the format of

**[Pod IP separated by dashes].[Namespace].[type].[Base Domain Name]**

Where `[type]` is `pod` in this example, put services can be resolved by the same convention.

For example:

```shell
/ # nslookup 10-42-2-68.default.pod.cluster.local
Server: 10.43.0.10
Address: 10.43.0.10:53

Name: 10-42-2-68.default.pod.cluster.local
Address: 10.42.2.68

```

Services follow a similar pattern

**[Service Name].[Namespace].[type].[Base Domain Name]**

For example:

```shell
my-svc.my-namespace.svc.cluster-domain.example
```

Headless services are those without a cluster ip, but will respond with a list of IP’s of pods that are applicable at that particular moment in time.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-headless
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-headless
```

We can modify the default behavior of the pod dns configuration in the yaml file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 8.8.8.8
    searches:
      - ns1.svc.cluster.local
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

CoreDNS also has a configmap that can be modified:

```shell
kubectl get cm coredns -n kube-system -o yaml                                                            
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        hosts /etc/coredns/NodeHosts {
          reload 1s
          fallthrough
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
  NodeHosts: |
    172.16.10.100 k3s-ranch-node-1
    172.16.10.101 k3s-ranch-node-2
    172.16.10.102 k3s-ranch-node-3

```

The Corefile configuration includes the following plugins of CoreDNS:

* `errors`: Errors are logged to stdout.
* `health`: Health of CoreDNS is reported to `http://localhost:8080/health`. In this extended syntax lameduck will make the process unhealthy then wait for 5 seconds before the process is shut down.
* `ready`: An HTTP endpoint on port 8181 will return 200 OK, when all plugins that are able to signal readiness have done so.
* `kubernetes`: CoreDNS will reply to DNS queries based on IP of the services and pods of Kubernetes. You can find more details about that plugin on the CoreDNS website. ttl allows you to set a custom TTL for responses. The default is 5 seconds. The minimum TTL allowed is 0 seconds, and the maximum is capped at 3600 seconds. Setting TTL to 0 will prevent records from being cached. The pods insecure option is provided for backward compatibility with kube-dns. You can use the pods verified option, which returns an A record only if there exists a pod in same namespace with matching IP. The pods disabled option can be used if you don't use pod records.
* `prometheus`: Metrics of CoreDNS are available at `http://localhost:9153/metrics` in Prometheus format (also known as OpenMetrics).
* `forward`: Any queries that are not within the cluster domain of Kubernetes will be forwarded to predefined resolvers (/etc/resolv.conf). cache: This enables a frontend cache.
* `loop`: Detects simple forwarding loops and halts the CoreDNS process if a loop is found.
* `reload`: Allows automatic reload of a changed Corefile. After you edit the ConfigMap configuration, allow two minutes for your changes to take effect.

You can modify the default CoreDNS behaviour by modifying the ConfigMap.

!!! success "Exam Tip"

    Understand how Pods and Services automatically have A/AAAA records populated and how to test lookups

!!! success "Exam Tip"

    A lot of CoreDNS's functionality is encapsulated in its `configmap`
