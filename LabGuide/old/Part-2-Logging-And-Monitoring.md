

**<span style="text-decoration:underline;">CKA Curriculum Lab Part 2 - Logging and Monitoring</span>**

**<span style="text-decoration:underline;">Requirements</span>**



*   A preconstructed Kubernetes lab with kubectl

**<span style="text-decoration:underline;">Guidelines</span>**

Before you begin:



*   Open up a ssh connection so you can run “kubectl” commands
*   Open up a browser window to [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/)

During the activity

Leverage [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/) as much as you need

**<span style="text-decoration:underline;">Lab Activity 1 - General cluster status</span>**

Leveraging a single kubectl command acquire the current status for the Scheduler, Controller-Manager and etcd components

**<span style="text-decoration:underline;">Lab Activity 2 - Acquiring component logs</span>**

List the logs from the following components on a master node:



*   Kube-APIServer
*   Kube-Scheduler
*   Kube-Controller-Manager

List the logs from the following components on a worker node:



*   CNI
*   Kube-Proxy
*   Kubelet
*   Container Runtime

**<span style="text-decoration:underline;">Lab Activity 3 - Application Monitoring</span>**

Run the following:


```
kubectl apply -f https://raw.githubusercontent.com/David-VTUK/CKAExampleYaml/master/nginx-svc-and-deployment.yaml

```



*   Gather the events from a given pod from this deployment
*   Gather the events from the replicaset created from this deployment
*   Gather the events from the service from this deployment
    *   Acquire the list of endpoints associated with this service
    *   How are the endpoints determined?
*   Which kubectl command would display all the logs from the pods created from this deployment?
