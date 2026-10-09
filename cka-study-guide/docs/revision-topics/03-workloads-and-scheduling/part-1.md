# Understand application deployments and how to perform rolling update and rollbacks

## Application Deployments

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
        image: nginx:1.31.5
        ports:
        - containerPort: 80
```

The main reason why leverage `deployments` is to manage a number of identical pods via one administrative unit - the `deployment` object. Should we need to make changes, we apply this to the `deployment` object, not individual pods. Because of the declarative nature of `deployments`, Kubernetes will rectify any changes between desired and running state, and rectify accordingly. For example, if we manually deleted.

We can then describe it with `kubectl describe deployment nginx-deployment`

## Rolling Updates

To update an existing deployment, we have two main options:

* Rolling Update
* Recreate

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
        image: nginx:nginx:1.31.6
        ports:
        - containerPort: 80
```

We can then apply this file `kubectl apply -f updateddeployment.yaml --record=true`

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
1     <none>
2     <none>
4     <none>
5     kubectl apply --filename=updateddeployment.yaml --record=true
```

Alternatively, we can also do this imperatively:

```shell
kubectl --record deployments/nginx-deployment set image deployments/nginx-deployment nginx=nginx:1.31.6

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

!!! success "Exam Tip"

    Standalone Pods (not deployed via a deployment object) will not get rescheduled when deleted. Pods from a deployment object, will, however.

!!! success "Exam Tip"

    Avoid deploying standalone Pods. If you only need 1 replica of a instance, deploy a deployment object with a single replica

!!! success "Exam Tip"

    Changes made to a deployment object will be reflected in the Pods that it deploys.
