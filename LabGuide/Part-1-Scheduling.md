

**<span style="text-decoration:underline;">CKA Lab Part 1 - Scheduling</span>**

**<span style="text-decoration:underline;">Requirements</span>**



*   A preconstructed Kubernetes lab with kubectl

**<span style="text-decoration:underline;">Guidelines</span>**

Before you begin:



*   Open up a ssh connection so you can run “kubectl” commands
*   Open up a browser window to [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/)

During the activity



*   Leverage [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/) as much as you need

**<span style="text-decoration:underline;">Lab Activity 1 - Label Selectors</span>**

Deploy two pods:

One pod with a label of “Tier = Web”

One pod with a label of “Tier = App”

Use any container image you see fit.

Verify the labels are applied.

**<span style="text-decoration:underline;">Lab Activity 2 - Daemonsets</span>**

Deploy a Daemonset that leverages the nginx image

Verify the daemonset has been created successfully

**<span style="text-decoration:underline;">Lab Activity 3 - Resource Limits</span>**

Create a new namespace called “Tenant-B-100Mi”

Create a memory limit of 100Mi for this namespace

Create a pod with a memory request of 150Mi, ensure the limit has been set by verifying you get a error message.

 **<span style="text-decoration:underline;">Lab Activity 4 - Multiple Schedulers</span>**

Assume another scheduler “CustomScheduler” has been created in your environment. Configure a pod to use this scheduler.

Validate the pod is using this scheduler.

**<span style="text-decoration:underline;">Lab Activity 5 - Schedule Pod without a scheduler</span>**

On one of the worker nodes:

Create the directory /etc/staticpods

Create a pod manifest file in this directory

Configure the kubelet service on this worker node to create pods from /etc/staticpods

**<span style="text-decoration:underline;">Lab Activity 6 - Display Scheduler Events</span>**

Create a pod manifest file using the nginx image which will create a pod called “nginx-web” (Alternatively do this via `kubectl run`)

Extract the events from the cluster, particularly those pertaining to scheduling to find where this pod was scheduled.

Extract the logs from the pod running the default scheduler, or from the respective file if running as a deamon service on your master node.

**<span style="text-decoration:underline;">Lab Activity 7 - Know how to configure the Kubernetes Scheduler</span>**

Configure the Kube-Scheduler by adding `--logtostderr=true` to the existing configuration.
