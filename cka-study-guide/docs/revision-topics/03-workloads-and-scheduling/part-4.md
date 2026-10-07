# Understand the primitives used to create robust, self-healing, application deployments

Deployments facilitate this by employing a reconciliation loop to check the number of deployed pods matches what’s defined in the manifest. Under the hood, deployments leverage ReplicaSets, which are primarily responsible for this feature.

Stateful Sets are similar to deployments, for example they manage the deployment and scaling of a series of pods. However, in addition to deployments they also provide guarantees about the ordering and uniqueness of Pods. A StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

StatefulSets are valuable for applications that require one or more of the following.

* Stable, unique network identifiers.
* Stable, persistent storage.
* Ordered, graceful deployment and scaling.
* Ordered, automated rolling updates.

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