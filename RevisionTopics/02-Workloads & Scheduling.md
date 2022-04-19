# Understand deployments and how to perform rolling update and rollbacks

Deployments are intended to replace Replication Controllers.  They provide the same replication functions (through Replica Sets) and also the ability to rollout changes and roll them back if necessary. An example configuration is shown below:

```yaml
apiVersion: apps/v1
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

The main reason why leverage `deployments` is to manage a number of identical pods via one administrative unit - the `deployment` object. Should we need to make changes, we apply this to the `deployment` object, not individual pods. Because of the declarative nature of `deployments`, Kubernetes will rectify any changes between desired and running state, and rectify accordingly. For example, if we manually deleted 

We can then describe it with` kubectl describe deployment nginx-deployment`

To update an existing deployment, we have two main options:

*   Rolling Update
*   Recreate

A rolling update, as the name implies, will swap out containers in a deployment with one created by a new image.

Use a rolling update when the application supports having a mix of different pods (aka application versions). This method will also involve no downtime of the service, but will take longer to bring up the deployment to the requested version. Old and new versions of the pod spec will coexist until they're all rotated.

A recreation will delete all the existing pods and then spin up new ones. This method will involve downtime. Consider this a “bing bang” approach

Examples listed in the Kubernetes documentation are largely imperative, but I prefer to be declarative. As an example, create a new yaml file and make the required changes, in this example, the version of the nginx container is incremented.


```yaml
apiVersion: apps/v1
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


```shell
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


```shell
kubectl rollout history deployment/nginx-deployment

deployment.extensions/nginx-deployment
REVISION  CHANGE-CAUSE
1     	<none>
2     	<none>
4     	<none>
5     	kubectl apply --filename=updateddeployment.yaml --record=true
```

Alternatively, we can also do this imperatively:

```shell
kubectl --record deployments/nginx-deployment set image deployments/nginx-deployment nginx=nginx:1.9.1

deployment.extensions/nginx-deployment image updated
deployment.extensions/nginx-deployment image updated
```

## Rollback

To rollback to the previous version:

```shell
kubectl rollout undo deployment/nginx-deployment 
```

To rollback to a specific version:

```shell
kubectl rollout undo deployment/nginx-deployment --to-revision 5
```

Source of `revision`: `kubectl rollout history deployment/nginx-deployment`

# Use ConfigMaps and Secrets to configure applications

Configmaps are a way to decouple configuration from a pod manifest. Obviously, the first step is to create a config map before we can get pods to use them:

```shell
kubectl create configmap <map-name> <data-source>
```

“Map-name” is an arbitrary name we give to this particular map, and “data-source” corresponds to a key-value pair that resides in the config map.


```shell
kubectl create configmap vt-cm --from-literal=blog=virtualthoughts.co.uk
```

At which point we can then describe it:

```shell
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

Configmaps can be mounted as `volumes` or `environment variables`. The below example leverages the latter.

```yaml
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


```shell
kubectl logs config-test-pod | grep "BLOG_NAME="
...
BLOG_NAME=virtualthoughts.co.uk
...
```

#  Know how to scale applications

Constantly adding more, individual pods is not a sustainable model for scaling an application. To facilitate applications at scale, we need to leverage higher level constructs such as replicasets or deployments. As mentioned previously, `deployments` provide us with a single administrative unit to manage the underlying pods. We can scale a `deployment` object to increase the number of `pods`.

As an example, if the following is deployed:

```yaml
apiVersion: apps/v1
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

```shell
kubectl scale deployment nginx-deployment --replicas 10
```

# Understand the primitives used to create robust, self-healing, application deployments

Deployments facilitate this by employing a reconciliation loop to check the number of deployed pods matches what’s defined in the manifest. Under the hood, deployments leverage ReplicaSets, which are primarily responsible for this feature.

Stateful Sets are similar to deployments, for example they manage the deployment and scaling of a series of pods. However, in addition to deployments they also provide guarantees about the ordering and uniqueness of Pods. A StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

StatefulSets are valuable for applications that require one or more of the following.

*   Stable, unique network identifiers.
*   Stable, persistent storage.
*   Ordered, graceful deployment and scaling.
*   Ordered, automated rolling updates.

```yaml
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

# Understand how resource limits can affect Pod scheduling

At a namespace level, we can define resource limits. This enables a restriction in resources, especially helpful in multi-tenancy environments and provides a mechanism to prevent pods from consuming more resources than permitted, which may have a detrimental effect on the environment as a whole.

We can define the following:

Default memory / CPU **requests & limits** for a namespace

Minimum and Maximum memory / CPU **constraints** for a namespace

Memory/CPU **Quotas** for a namespace

## Default Requests and Limits

If a container is created in a namespace with a default request/limit value and doesn't explicitly define these in the manifest, it inherits these values from the namespace

Note, if you define a container with a memory/CPU limit, but not a request, Kubernetes will define the limit the same as the request.

## Minimum / Maximum Constraints

If a pod does not meet the range in which the constraints are valued at, it will not be scheduled.

## Quotas

Control the _total_ amount of CPU/memory that can be consumed in the _namespace_ as a whole.

Example: Attempt to schedule a pod that request more memory than defined in the namespace

Create a namespace:
```shell
kubectl create namespace tenant-mem-limited
```
Create a YAML manifest to limit resources:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: tenant-max-mem
  namespace: tenant-mem-limited
spec:
  limits:
  - max:
      memory: 250Mi
    type: Container
```

Apply this to the aforementioned namespace: 

```shell
kubectl apply -f maxmem.yaml`
```

To create a pod with a memory request that exceeds the limit:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: too-much-memory
  namespace: tenant-mem-limited 
spec:
  containers:
  - name: too-much-mem
    image: nginx
    resources:
      requests:
        memory: "300Mi"
```

Executing the above will yield the following result:

```shell
The Pod "too-much-memory" is invalid: spec.containers[0].resources.requests: Invalid value: "300Mi": must be less than or equal to memory limit
```

As we have defined the pod limit of the namespace to 250MiB, a request for 300MiB will fail.

#  Awareness of manifest management and common templating tools

## Kustomize

Kustomize is a templating tool for Kubernetes manifests in its native form (Yaml). When working with raw YAML files you will typically have a directory containing several files identifying the resources it creates. To begin, a directory containing our manifests needs to exist:

```shell
/home/david/app/base
total 16
drwxrwxr-x  2 david david 4096 Feb  9 11:44 .
drwxr-xr-x 27 david david 4096 Feb  9 11:44 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

This will form our `base` - we will build on this but adding customisations in the form of overlays. First, we need a `kustomize` file. which can be created with `kustomize create --autodetect`

This will create kustomization.yaml in the current directory:

```shell
total 20
drwxrwxr-x  2 david david 4096 Feb  9 11:47 .
drwxr-xr-x 27 david david 4096 Feb  9 11:47 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  108 Feb  9 11:47 kustomization.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

The contents being:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
```

## Variants and Overlays

* variant - Divergence in configuration from the `base`
* overlay - Composes variants together 

Say, for example, we wanted to generate manifests for different environments (prod and dev) that are based from this config, but have additional customisations. In this example we will create a `dev` variant encapsulated in a single Overlay

```shell
mkdir -p overlays/{dev,prod}
cd overlays/dev 
```
Begin by creating a Kustomization object specifying the base (this will create `kustomization.yaml`) :

```shell
kustomize create --resources ../../base
```
In this example, I want to change the replica count to 1, as it's a dev environment. In the `dev` directory, create a new file `deployment.yaml` containing:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
```

The `kustomization.yaml` file needs modifying to include a `patchesStrategicMerge` block:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
patchesStrategicMerge:
  - deployment.yaml
```
Patches can be used to apply different customizations to Resources. Kustomize supports different patching mechanisms through `patchesStrategicMerge` and `patchesJson6902`. `patchesStrategicMerge` is a list of file paths.

We can generate the manifests and apply to the cluster by executing (from the base folder):

```shell
kustomize build ./overlay/dev | kubectl apply -f -
```

By running this, only 1 pod will be created in the deployment object, instead of what's defined in the `base` because of the customisation we've applied. We can do the same with prod, or any arbitrary number of environments.

## Helm

Helm is synonymous to what `apt` or `yum` are in the Linux world. It's effectively a package manager for Kubernetes. "Packages" in Helm are called `charts` to which you can customise with your own values.

It's unlikely the exam will require anyone to create a helm chart from scratch, but an understanding of how it works is a good idea.

## Helm Repos

Repos are where helm charts are stored. Typically, a repo will contain a number of charts to choose from. Helm can be managed by a CLI client, and a repo can be added by running:

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

To list the packages from this repo:

```shell
helm search repo bitnami
```

To install a package from this repo:

```shell
helm install my-release bitnami/mariadb
```

Where `my-release` is a string identifying an installed instance of this application

Parameters that can be customised - are dependent on how the chart is configured. For the aforementioned MariaDB chart, they are listed at  https://github.com/bitnami/charts/tree/master/bitnami/mariadb/#parameters

These values are encapsulated in the corresponding `values.yaml` file in the repo. You can populate an instance of it and apply it with:

```shell
helm install -f https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mariadb/values.yaml my-release bitnami/mariadb
```

Alternatively, variables can be declared by using `--set`, such as:

```shell
helm install my-release --set auth.rootPassword=secretpassword bitnami/mariadb
```
