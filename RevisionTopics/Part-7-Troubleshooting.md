

**<span style="text-decoration:underline;">CKA Curriculum V1.12.0 Part 7 - Troubleshooting</span>**

**<span style="text-decoration:underline;">Troubleshoot application failure</span>**

This is largely dependent on the application in question and what’s being leveraged in terms of the Kubernetes infrastructure, but from a top level.

**Pods**

**<code>kubectl get pods </code></strong>- Get the pods currently running and their status

**<code>kubectl logs nginx-web </code></strong>- Get stderr/stdout from pod

**<code>kubectl describe pod nginx-web </code></strong>- Get detailed information about a pod, including the steps that were required to run it, for example pulling images, assigning to nodes, etc.


```
Events:
 Type    Reason     Age   From                    Message
 ----    ------     ----  ----                    -------
 Normal  Scheduled  7s    default-scheduler       Successfully assigned default/nginx-web to k8
S-worker-04

 Normal  Pulling    6s    kubelet, k8s-worker-04  Pulling image "nginx"
 Normal  Pulled     4s    kubelet, k8s-worker-04  Successfully pulled image "nginx"
 Normal  Created    4s    kubelet, k8s-worker-04  Created container nginx
 Normal  Started    3s    kubelet, k8s-worker-04  Started container nginx
```


**Services**


```
kubectl get services
kubectl describe service kubernetes
```


When diagnosing services, particularly those which are only accessible internally (the default service type) spin up a pod to which you can exec into to run tests.


```
kubectl run -i --tty busybox --image=busybox -- sh
```


At which point tools such as ping/telnet

**Deployments**

<span style="text-decoration:underline;">Kubectl get deployments</span>

<span style="text-decoration:underline;">Kubectl describe deployment nginx</span>

**<span style="text-decoration:underline;">Troubleshoot control plane failure</span>**


```
kubectl get componentstatuses

NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-0               Healthy   {"health":"true"} 
```


Refer to part 2 under “Manage cluster component logs” to identify the log locations/journalctl commands for the various k8s cluster components.

**<span style="text-decoration:underline;">Troubleshoot worker node failure</span>**

Refer to “CKA Curriculum Part 2 - Logging and Monitoring” for troubleshooting the controller components via log files or journalctl.

**<span style="text-decoration:underline;">Troubleshoot networking</span>**

Refer to “CKA Curriculum Part 2 - Logging and Monitoring” for troubleshooting the CNI.

In addition:



*   Spin up a pod for testing internal cluster networking (service IP)
*   Describe pods/services to ensure the correct endpoints are being added
*   Ensure pods/services are exposed on the correct port
*   Exec directly into pods and run commands locally
