# Monitor cluster and application resource usage

Several `kubectl` commands can be used to determine resource usage at the cluster level and at the application level:

## Cluster level resource usage

Use `kubectl top nodes` to view CPU and memory consumption per node:

```bash
ubuntu@srv-rk1-01:~$ kubectl top node
NAME         CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
srv-rk1-01   754m         9%       5777Mi          18%         
srv-rk1-02   535m         6%       3750Mi          11%         
srv-rk1-03   609m         7%       3086Mi          9%          
srv-rk1-04   534m         6%       4301Mi          13%   
```

## Application level resource usage

In modern environments we would usually have access to some kind of monitoring stack, but for the purposes of the CKA exam, we'll use what's in minimal Kubernetes distributions. Therefore, similarly with the command above we can leverage `kubectl top pod -A` to list the top pods from all namespaces:

```bash
ubuntu@srv-rk1-01:~$ kubectl top pod -A
NAMESPACE                NAME                                                        CPU(cores)   MEMORY(bytes)   
argocd                   argocd-application-controller-0                             4m           519Mi           
argocd                   argocd-applicationset-controller-65b78d8965-227jk           9m           39Mi            
argocd                   argocd-dex-server-7b6ccd69b6-h892j                          1m           18Mi            
argocd                   argocd-notifications-controller-5cbd56d77c-vw8kp            1m           24Mi            
argocd                   argocd-redis-7f9487d4fd-knhpn                               5m           10Mi            
argocd                   argocd-repo-server-75c8d77db5-7ltcz                         4m           78Mi            
argocd                   argocd-server-5b5fc4f986-658l2                              1m           25Mi            
cert-manager             cert-manager-58cbcbdbf4-n78v6                               3m           29Mi            
cert-manager             cert-manager-cainjector-6468bc96c7-rwp7m                    2m           64Mi            
cert-manager             cert-manager-webhook-558c6d4f4d-5g8hc                       1m           12Mi            
```

By default this will iterate through all namespaces alphabetically. If we wanted to view the top pods by CPU usage we can run `kubectl top pods -A --sort-by=cpu`.

```bash
david@fedora:~/cka$ kubectl top pods -A --sort-by=cpu
NAMESPACE                NAME                                                        CPU(cores)   MEMORY(bytes)   
kube-prometheus-stack    prometheus-kube-prometheus-stack-prometheus-0               179m         924Mi           
cilium                   cilium-6bwkx                                                46m          451Mi           
kubevirt                 virt-operator-787d685f97-c6txf                              43m          248Mi           
cilium                   cilium-mw95j                                                43m          446Mi           
longhorn                 instance-manager-211d2259c96952e090fd2eb2f9c44f92           42m          152Mi           
cilium                   cilium-kfh7p                                                40m          441Mi           
```

And for RAM: `kubectl top pods -A --sort-by=memory`

```bash
david@fedora:~/cka$ kubectl top pods -A --sort-by=memory
NAMESPACE                NAME                                                        CPU(cores)   MEMORY(bytes)   
kube-prometheus-stack    prometheus-kube-prometheus-stack-prometheus-0               39m          933Mi           
argocd                   argocd-application-controller-0                             123m         550Mi           
kube-prometheus-stack    kube-prometheus-stack-grafana-69fdf74c6f-bxpws              16m          459Mi           
cilium                   cilium-6bwkx                                                51m          451Mi           
cilium                   cilium-mw95j                                                43m          446Mi           
cilium                   cilium-qzj5k                                                44m          442Mi           
cilium                   cilium-kfh7p                                                41m          442Mi           
kubevirt                 virt-operator-787d685f97-c6txf                              22m          244Mi   
```

We can also check a Pods configured memory limits by inspecting its configuration:

```bash
kubectl describe pod <pod-name> | grep -A 4 Limits
kubectl describe pod <pod-name> | grep -A 4 Requests
```

!!! success "Exam Tip"

    If you see a Pod has been `OOM Killed` (Out of Memory Killed) but the node it runs on has available resources - check the Pods memory limit. If a Pod tries to exceed its memory limit, it will be `OOM Killed`.
