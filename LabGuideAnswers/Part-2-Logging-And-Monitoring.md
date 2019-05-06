
**<span style="text-decoration:underline;">CKA Curriculum Lab Part 2 - Logging and Monitoring Answers</span>**

**<span style="text-decoration:underline;">Lab Activity 1 - General cluster status</span>**

Leveraging a single kubectl command acquire the current status for the Scheduler, Controller-Manager and etcd components


```
kubectl get componentstatuses
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health":"true"}   
```


**<span style="text-decoration:underline;">Lab Activity 2 - Acquiring component logs</span>**

List the logs from the following components on a master node:



*   Kube-APIServer

**First activity is to locate the logs, be it from a pod/log file/or systemd unit**


```
cat /var/log/kube-apiserver.log
or
journalctl -u kube-apiserver
or
kubectl logs kube-apiserver-k8s-master-03 -n kube-system

```



*   Kube-Scheduler


```
cat /var/log/kube-scheduler.log
or
journalctl -u kube-scheduler
or
kubectl logs kube-scheduler-k8s-master-03 -n kube-system

```



*   Kube-Controller-Manager


```
cat /var/log/kube-controller-manager.log
or
journalctl -u kube-controller-manager
or
kubectl logs kube-controller-manager-k8s-master-03 -n kube-system
```


List the logs from the following components on a worker node:



*   CNI

**Flannel as an example:**


```
journalctl -u flanneld
```


**or**


```
kubectl logs kube-flannel-ds-amd64-9mrnj -n kube-system

```



*   Kube-Proxy


```
journalctl -u kube-proxy
or
cat /var/log/kube-proxy.log
or
kubectl logs kube-flannel-ds-amd64-9mrnj -n kube-system

```



*   Kubelet


```
journalctl -u kubelet
Or
cat /var/log/kubelet.log

```



*   Container Runtime


```
journalctl -u docker.service
Or
cat /var/log/docker.log
```


**<span style="text-decoration:underline;">Lab Activity 3 - Application Monitoring</span>**

Run the following:


```
kubectl apply -f https://raw.githubusercontent.com/David-VTUK/CKAExampleYaml/master/nginx-svc-and-deployment.yaml

```



*   Gather the events from a given pod from this deployment


```
kubectl get pods
kubectl describe pod nginx-deployment-6dd86d77d-7tm22

```



*   Gather the events from the replicaset created from this deployment


```
kubectl describe replicaset nginx-deployment-6dd86d77d

```



*   Gather the events from the service from this deployment


```
kubectl describe svc nginx-service

```



    *   Acquire the list of endpoints associated with this service


```
Endpoints:         10.244.1.73:80,10.244.1.74:80,10.244.2.53:80

```



    *   How are the endpoints determined?


```
Selector:          app=nginx

```



*   Which kubectl command would display all the logs from the pods created from this deployment?


```
kubectl logs -l app=nginx --all-containers=true
