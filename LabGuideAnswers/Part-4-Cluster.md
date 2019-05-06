

**<span style="text-decoration:underline;">CKA Lab Part 4 - Cluster</span>**

**<span style="text-decoration:underline;">Lab 1 - Cluster Upgrades</span>**

Upgrade a Kubernetes cluster running a previous version to the latest version



*   Upgrade the kubeadm binary


```
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.13.x-00 && \
apt-mark hold kubeadm

```



*   Assess the upgrade path that kubeadm has provided


```
sudo kubeadm upgrade plan

```



*   Note down the command to execute the upgrade


```
sudo kubeadm upgrade apply v1.14.1

```



*   Upgrade Kubelet on all the nodes


```
apt-mark unhold kubelet && \
apt-get update && apt-get install -y kubelet=1.13.x-00 && \
apt-mark hold kubelet
```


**<span style="text-decoration:underline;">Lab 2 - Cluster Upgrades - OS Upgrades</span>**



*   **Gracefully** remove a node from active service


```
kubectl drain k8s-worker-03 --ignore-daemonsets

node/k8s-worker-03 cordoned
WARNING: Ignoring DaemonSet-managed pods: kube-flannel-ds-amd64-9mrnj, kube-proxy-fq4vx

pod/nginx-deployment-6dd86d77d-vbjcl evicted
pod/nginx-deployment-6dd86d77d-hsthw evicted

node/k8s-worker-03 evicted

```



    *   Verify the node has scheduling disabled


```
kubectl get nodes
NAME            STATUS                     ROLES    AGE   VERSION
k8s-master-03   Ready                      master   14d   v1.14.1
k8s-worker-03   Ready,SchedulingDisabled   <none>   14d   v1.14.1
k8s-worker-04   Ready                      <none>   14d   v1.14.1

```



*   Gracefully return a node into active service


```
kubectl uncordon k8s-worker-03                     
node/k8s-worker-03 uncordoned

```



    *   Verify the node has scheduling enabled 


```
kubectl get nodes              
NAME            STATUS   ROLES    AGE   VERSION
k8s-master-03   Ready    master   14d   v1.14.1
k8s-worker-03   Ready    <none>   14d   v1.14.1
k8s-worker-04   Ready    <none>   14d   v1.14.1
```


**<span style="text-decoration:underline;">Lab 3 - Back up etcd</span>**



*   Take a snapshot of the etcd database


```
sudo ETCDCTL_API=3 etcdctl snapshot save snapshotdb 
--cacert /etc/kubernetes/pki/etcd/server.crt 
--cert /etc/kubernetes/pki/etcd/ca.crt 
--key /etc/kubernetes/pki/etcd/ca.key

```



*   Verify the snapshot


```
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshotdb
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 45c3ef38 |  1851970 |       1052 |     3.8 MB |
+----------+----------+------------+------------+
```


**<span style="text-decoration:underline;">Lab 4 - Back up Kubernetes certificates</span>**



*   Tarball the k8s certificate directory


```
cd /etc/kubernetes/pki/
ls -la
total 84
drwxr-xr-x 3 root root 4096 Apr 16 08:31 .
drwxr-xr-x 5 root root 4096 Apr 12 13:45 ..
-rw-r--r-- 1 root root 1229 Apr 11 15:23 apiserver.crt
-rw-r--r-- 1 root root 1090 Apr 12 13:45 apiserver-etcd-client.crt
-rw------- 1 root root 1679 Apr 12 13:45 apiserver-etcd-client.key
-rw------- 1 root root 1679 Apr 11 15:23 apiserver.key
-rw-r--r-- 1 root root 1099 Apr 11 15:23 apiserver-kubelet-client.crt
-rw------- 1 root root 1675 Apr 11 15:23 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1025 Apr 11 15:23 ca.crt
-rw------- 1 root root 1675 Apr 11 15:23 ca.key
-rw-r--r-- 1 root root   17 Apr 16 08:31 ca.srl
drwxr-xr-x 2 root root 4096 Apr 11 15:23 etcd
-rw-r--r-- 1 root root 1038 Apr 11 15:23 front-proxy-ca.crt
-rw------- 1 root root 1675 Apr 11 15:23 front-proxy-ca.key
-rw-r--r-- 1 root root 1058 Apr 11 15:23 front-proxy-client.crt
-rw------- 1 root root 1679 Apr 11 15:23 front-proxy-client.key
-rw------- 1 root root 1675 Apr 11 15:23 sa.key
-rw------- 1 root root  451 Apr 11 15:23 sa.pub
-rw-r--r-- 1 root root 1005 Apr 16 08:31 vt.crt
-rw-r--r-- 1 root root  915 Apr 16 08:30 vt.csr
-rw------- 1 root root 1675 Apr 16 08:30 vt.key

Tar -zcvf pki-certs.tar.gz /etc/kubernetes/pki
