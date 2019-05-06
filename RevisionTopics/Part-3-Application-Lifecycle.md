
**<span style="text-decoration:underline;">CKA Curriculum V1.12.0 Part 3 - Application Lifecycle Management</span>**

**<span style="text-decoration:underline;">Understand deployments and how to perform rolling updates and rollbacks</span>**

Deployments are intended to replace Replication Controllers.  They provide the same replication functions (through Replica Sets) and also the ability to rollout changes and roll them back if necessary. An example configuration is shown below:


```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```


We can then describe it with` kubectl describe deployment nginx-deployment`

To update an existing deployment, we have two main options:



*   Rolling Update
*   Recreate

A rolling update, as the name implies, will swap out containers in a deployment with one created by a new image.

Use a rolling update when the application supports having a mix of different pods (aka application versions). This method will also involve no downtime of the service, but will take longer to bring up the deployment to the requested version

A recreate will delete all the existing pods and then spin up new ones. This method will involve downtime. Consider this a “bing bang” approach

Examples listed in the Kubernetes documentation are largely imperative, but I prefer to be declarative. As an example, create a new yaml file and make the required changes, in this example, the version of the nginx container is incremented.


```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.15
        ports:
        - containerPort: 80
```


We can then apply this file` kubectl apply -f updateddeployment.yaml --record=true`

Followed by the following:


```
kubectl rollout status deployment/nginx-deployment

Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 4 of 5 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
```


We can also use the kubectl rollout history to look at the revision history of a deployment


```
kubectl rollout history deployment/nginx-deployment

deployment.extensions/nginx-deployment
REVISION  CHANGE-CAUSE
1     	<none>
2     	<none>
4     	<none>
5     	kubectl apply --filename=updateddeployment.yaml --record=true
```


Alternatively, we can also do this imperatively:


```
kubectl --record deployments/nginx-deployment set image deployments/nginx-deployment nginx=nginx:1.9.1

deployment.extensions/nginx-deployment image updated
deployment.extensions/nginx-deployment image updated
```


<span style="text-decoration:underline;">Rollback</span>

To rollback to the previous version:


```
kubectl rollout undo deployment/nginx-deployment 
```


To rollback to a specific version:


```
kubectl rollout undo deployment/nginx-deployment --to-revision 5
```


Where revision number comes from `kubectl rollout history deployment/nginx-deployment`

**<span style="text-decoration:underline;">Know various ways to configure applications</span>**

I personally find this subject a bit ambiguous, but we can attempt to break this down into the following objet types:

**Jobs**

**Configmaps**

**Secrets**

**Environment variables**

**<span style="text-decoration:underline;">Jobs</span>**

A job is  simply an application that runs to completion within a pod. Use case for this would be something like batch processing.

The example below executes a perl command inside a container that calculates pi to 2000 places, printing out the result. After it has completed, the pod will terminate.


```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```


We can check the status of the job with the following:


```
kubectl get jobs

NAME   COMPLETIONS   DURATION   AGE
pi     1/1           97s        102s
```


And the respective container:


```
kubectl get pods
NAME       READY   STATUS      RESTARTS   AGE
pi-vjk5m   0/1     Completed   0          2m54s
```


To view the output from this pod:


```
kubectl logs pi-vjk5m

3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359………..
```


**Config maps**

Configmaps are a way to decouple configuration from pod manifest file. Obviously, the first step is to create a config map before we can get pods to use them:


```
kubectl create configmap <map-name> <data-source>
```


“Map-name” is a arbitrary name we give to this particular map, and “data-source” corresponds to a key-value pair that resides in the config map.


```
kubectl create configmap vt-cm --from-literal=blog=virtualthoughts.co.uk
```


At which point we can then describe it:


```
kubectl describe configmap vt-cm
Name:         vt-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
blog:
----
virtualthoughts.co.uk
```


To reference this config map in a pod, we declare it in the respective yaml:


```
apiVersion: v1
kind: Pod
metadata:
 name: config-test-pod
spec:
 containers:
 - name: test-container
   image: busybox
   command: [ "/bin/sh", "-c", "env" ]
   env:
     - name: BLOG_NAME
       valueFrom:
         configMapKeyRef:
           name: vt-cm
           key: blog
 restartPolicy: Never
```


The pod above will output the environment variables, so we can validate it’s leveraged the config map by extracting the logs from the pod:


```
kubectl logs config-test-pod
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=config-test-pod
SHLVL=1
HOME=/root
BLOG_NAME=virtualthoughts.co.uk
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```


**Secrets**

Secrets allow us to store and manage sensitive information pertaining to our applications, which can take the form of a variety of objects such as usernames, passwords, ssh keys and much more. Similarly to configmaps, secrets are designed to decouple this information directly from the pod declaration.

As part of a Kubernetes cluster stand up, secrets are already leveraged, and we can view them by executing the following:


```
kubectl get secrets --all-namespaces
```


An example to create one:


```
kubectl create secret generic my-secret --from-literal=username=dbu --from-literal=pass=dbp
```


We leverage secrets in a very similar way to configmaps. (note as environment variables is one of a number of ways to leverage secrets)


```
apiVersion: v1
kind: Pod
metadata:
 name: secret-test-pod
spec:
 containers:
 - name: test-container
   image: busybox
   command: [ "/bin/sh", "-c", "env" ]
   env:
     - name: DB_Username
       valueFrom:
         secretKeyRef:
           name: my-secret
           key: username
     - name: DB_Pass
       valueFrom:
         secretKeyRef:
           name: my-secret
           key: pass
 restartPolicy: Never

kubectl logs secret-test-pod
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=secret-test-pod
DB_Username=dbu
SHLVL=1
HOME=/root
DB_Pass=dbp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```


**Environment Variables**

For more simple, direct configuration we can manipulate the environment variables directly within the pod manifest:


```
apiVersion: v1
kind: Pod
metadata:
 name: secret-test-pod
spec:
 containers:
 - name: test-container
   image: busybox
   command: [ "/bin/sh", "-c", "env" ]
   env:
     - name: DB_Username
       value:  "some username"
     - name: DB_Pass
       Value: "some password"
 restartPolicy: Never
```


**<span style="text-decoration:underline;">Know how to Scale Applications</span>**

Constantly adding more, individual pods is not a sustainable model for scaling an application. To facilitate applications at scale, we need to leverage higher level constructs such as replicasets or deployments.

As an example, if the following is deployed:


```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
 name: nginx-deployment
spec:
 replicas: 5
 template:
   metadata:
     labels:
       app: nginx-frontend
   spec:
     containers:
     - name: nginx
       image: nginx:1.14
       ports:
       - containerPort: 80
```


If we wanted to scale this, we can simply modify the yaml file and scale up/down the deployment by modifying the “replicas” field, or modify it in the fly:


```
kubectl scale deployment nginx-deployment --replicas 10
```


**<span style="text-decoration:underline;">Understand the primitives necessary to create a self-healing application</span>**

Deployments facilitate this by employing a reconciliation loop to check the number of deployed pods matches what’s defined in the yaml file. Under the hood, deployments leverage ReplicaSets, which are primarily responsible for this feature. 

Stateful Sets are similar to deployments, for example they manage the deployment and scaling of a series of pods. However, in addition to deployments they also provide guarantees about the ordering and uniqueness of Pods. A StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

StatefulSets are valuable for applications that require one or more of the following.



*   Stable, unique network identifiers.
*   Stable, persistent storage.
*   Ordered, graceful deployment and scaling.
*   Ordered, automated rolling updates.


```
apiVersion: apps/v1
kind: StatefulSet
metadata:
 name: nginx-statefulset
spec:
 selector:
   matchLabels:
     app: vt-nginx
 serviceName: "nginx"
 replicas: 2
 template:
   metadata:
     labels:
       app: vt-nginx
   spec:
     containers:
     - name: vt-nginx
       image: nginx:1.7.9
       ports:
       - containerPort: 80
```


**<span style="text-decoration:underline;">Understand the Kubernetes cluster upgrade process</span>**

Kubeadm is likely going to be the tool of choice to upgrade a Kubernetes cluster. Firstly, define the version you wish to upgrade to, or find the latest stable release:


```
export VERSION=$(curl -sSL https://dl.k8s.io/release/stable.txt)
export ARCH=amd64
```


Download the latest version of kubeadm


```
curl -sSL https://dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubeadm > kubeadm
```


Install Kubeadm


```
sudo install -o root -g root -m 0755 ./kubeadm /usr/bin/kubeadm
```


Then, execute a upgrade plan


```
sudo kubeadm upgrade plan

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.13.0   v1.14.1

Upgrade to the latest stable version:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.13.5   v1.14.1
Controller Manager   v1.13.5   v1.14.1
Scheduler            v1.13.5   v1.14.1
Kube Proxy           v1.13.5   v1.14.1
CoreDNS              1.2.6     1.3.1
Etcd                 3.2.24    3.3.10

You can now apply the upgrade by executing the following command:

       kubeadm upgrade apply v1.14.1

As the output implies, execute the kubeadm upgrade command (under root/sudo) to apply the changes

linu
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/version] You have chosen to change the cluster version to "v1.14.1"
[upgrade/versions] Cluster version: v1.13.5
[upgrade/versions] kubeadm version: v1.14.1
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
[upgrade/prepull] Prepulling image for component etcd.
```


After which, kubeadm will start pulling down updated container images and deploy those to your cluster.

If you run a kubectl get nodes on your cluster following a kubeadm upgrade apply command, you’ll notice that the version listed hasn’t changed:


```
NAME            STATUS   ROLES    AGE   VERSION
k8s-master-03   Ready    master   22h   v1.13.0
k8s-worker-03   Ready    <none>   22h   v1.13.0
k8s-worker-04   Ready    <none>   22h   v1.13.0
```


Recall from the kubeadm upgrade command:


```
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.13.0   v1.14.1
```


The version number from kubectl get nodes refers to the kubelet version. As the tool implies, you need to upgrade this manually.

The following needs to be done on each node. Note the kubelet service needs restarting after the new binary has been implemented.


```
export VERSION=$(curl -sSL https://dl.k8s.io/release/stable.txt)
export ARCH=amd64
curl -sSL https://dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubelet > kubelet
sudo install -o root -g root -m 0755 ./kubelet /usr/bin/kubelet
sudo systemctl restart kubelet.service
```


**<span style="text-decoration:underline;">Facilitate Operating System Upgrades</span>**

Occasionally upgrades have to be made to the underlying host that’s facilitating your pods. To do this in a graceful way, we can do the following.

Get a list of nodes running on which node


```
kubectl get pods -o wide

Drain the pods from the node:

kubectl drain k8s-worker04 --ignore-daemonsets

node/k8s-worker-04 cordoned
WARNING: Ignoring DaemonSet-managed pods: kube-flannel-ds-amd64-hq4wh, kube-proxy-cqpst
pod/nginx-7db9fccd9b-tdpx7 evicted
pod/coredns-fb8b8dccf-hj5km evicted
node/k8s-worker-04 evicted
```


As we can see, the non daemonset pods have been evicted and the host has been cordoned. This will apply the “SchedulingDisabled” flag on the node


```
kubectl get nodes
NAME        	STATUS                	 
k8s-worker-04   Ready,SchedulingDisabled
```


At this point, complete the required maintenance activities. Even if you reboot, the node will still be in a “SchedulingDisabled” state.

To release the node back to scheduling duties, uncordon the node:


```
kubectl uncordon k8s-worker-04
```


At which point it will be ready:


```
kubectl get nodes
NAME        	STATUS                	 
k8s-worker-04   Ready
```


**<span style="text-decoration:underline;">Implement backup and restore methodologies</span>**

In a Kubernetes cluster there are two main pieces of data that need backing up:



*   Certificate Files
*   Etcd database

**Backing up certificate files**

Back up the /etc/kubernets/pki directory. This can also be done as a cronjob, and contains the following files:


```
/etc/kubernetes/pki$ ls
apiserver.crt          	
apiserver.key             	
ca.crt  
front-proxy-ca.crt  	
front-proxy-client.key
apiserver-etcd-client.crt  
apiserver-kubelet-client.crt  
ca.key  
front-proxy-ca.key  	
sa.key
apiserver-etcd-client.key  
apiserver-kubelet-client.key  
etcd	front-proxy-client.crt  
Sa.pub
```


**Backing up etcd**

**Take a snapshot of the DB, then store it in a safe location**


```
ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
```


Verify the backup:


```
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 2125d542 |   364069 |        770 |  	3.8 MB |
+----------+----------+------------+------------+
