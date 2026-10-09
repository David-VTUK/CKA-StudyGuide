# Troubleshoot Services and Networking

## DNS Resolution

`Pods` and `Services` will automatically have a DNS record registered against `coredns` in the cluster, aka "A" records for IPv4 and "AAAA" for IPv6. The format of which is:

`pod-ip-address.namespace.pod.cluster-domain`
`my-svc-name.namespace.svc.cluster-domain`

Pod DNS records resolve to a single entity, even if the Pod contains multiple containers as they share the same networking namespace.

Service DNS records resolve to the respective service object.

Pods will automatically have their DNS resolution configured based on the clusters coredns settings. This can be validated by opening a shell to the pod and inspecting /etc/resolv.conf:

```shell
> kubectl exec -it web-server sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
/ # cat /etc/resolv.conf 
nameserver 10.43.0.10
search default.svc.cluster.local svc.cluster.local cluster.local eu-central-1.compute.internal
options ndots:5
```

`10.43.0.10` being the coredns service object:

```shell
> kubectl get svc -n kube-system 
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
kube-dns                     ClusterIP   10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP         16d
```

To test resolution, we can run a pod with `nslookup` to test. For the pod below:

```shell
> kubectl get po -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
web-server   1/1     Running   0          2d20h   10.42.1.31   ip-172-31-36-67   <none>           <none>
```

And knowing the format of the A record:

`pod-ip-address.my-namespace.pod.cluster-domain.example`

We should be able to resolve `10-42-1-31.default.pod.cluster.local`. Tip : To determine the cluster domain, inspect the coredns configmap. Below indicating `cluster.local`.

```shell
> kubectl get cm coredns -n kube-system -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
          lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
```

Create a Pod with the tools required:

```shell
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

Test lookup:

```shell
kubectl exec -i -t dnsutils -- nslookup 10-42-1-31.default.pod.cluster.local
```

```shell
> kubectl exec -i -t dnsutils -- nslookup 10-42-1-31.default.pod.cluster.local
Server:         10.43.0.10
Address:        10.43.0.10#53

Name:   10-42-1-31.default.pod.cluster.local
Address: 10.42.1.31
```

Similarly, for a service, in this case a service called `nginx-service` that resides in the default namespace:

```shell
> kubectl exec -i -t dnsutils -- nslookup nginx-service.default.svc.cluster.local
Server:         10.43.0.10
Address:        10.43.0.10#53

Name:   nginx-service.default.svc.cluster.local
Address: 10.43.0.223
```

```shell
> kubectl get svc
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.43.0.223   <none>        80/TCP    9m15s
```

## CNI Issues

Mainly covered earlier in acquiring logs for the CNI. However, one issue that might occur is when a CNI is incorrectly, or not initialised. This may cause workloads to enter a `pending` status:

```shell
kubectl get po -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     Pending   0          57s   <none>   <none>   <none>           <none>
```

`kubectl describe <pod>` can help identify issues with assigning IP addresses to nodes from the CNI.

## Port Checking

Similarly, with leveraging `nslookup` to validate DNS resolution in our cluster, we can lean on other tools to perform other diagnostic. All we need is a `Pod` that has a utility like `netcat`, `telnet` etc.

## Endpoint Checking

`kubectl get endpoints <service-name>` - Verify the service is successfully mapping to live pod IPs. If this is empty, your service selectors likely don't match your pod labels.

```bash
david@fedora:~/cka$ kubectl get endpoints argocd-server -n argocd
NAME            ENDPOINTS                         AGE
argocd-server   10.0.1.121:8080                   39d
```

## Bypass K8s Service

If you want to test connectivity by proxying the service to your local machine:

`kubectl port-forward svc/<service-name> LocalMachinePort:WorkloadPort` - Bypass ingress entirely to test HTTP service routing directly from your local machine.

```bash
david@fedora:~/cka$ kubectl port-forward svc/longhorn-frontend -n longhorn 8888:8000
Forwarding from 127.0.0.1:8888 -> 8000
Forwarding from [::1]:8888 -> 8000
# On local machine, curl localhost:8888
Handling connection for 8888
Handling connection for 8888
Handling connection for 8888
Handling connection for 8888
Handling connection for 8888
Handling connection for 8888
Handling connection for 8888
```

!!! success "Exam Tip"

    Lean on standard troubleshooting tools, `nslookup`, `dig`, `netcat`, `curl`, `wget` as you would do for non-containerised environments.

!!! success "Exam Tip"

    Section 5 goes through services in more detail, don't worry if it's still confusing at this stage.