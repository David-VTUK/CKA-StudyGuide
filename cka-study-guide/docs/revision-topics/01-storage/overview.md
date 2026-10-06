# Storage

Storage in Kubernetes enables the persistence of data independent of the lifecycle of the Pod. All Pods use some form of storage, and by default Pods will use `ephemeral` storage - a temporary, non persistent placeholder for data intrinsically tied to the lifecycle of the Pod; if the Pod terminates, its data is gone. If it gets re-scheduled (for example, in the event it's terminated) it will resort to the state of the image.

Let's take a practical example and deploy simple single replica `nginx` deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.31.6
          ports:
            - containerPort: 80
              protocol: TCP

```

Acquire Pod Name:

```bash
david@fedora:~/cka$ kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-8df5fbf9b-8m6f8   1/1     Running   0          69s
```

Exec into it and overwrite the default index.html:

```bash
david@fedora:~/cka$ kubectl exec -it nginx-deployment-8df5fbf9b-8m6f8 -- bash
root@nginx-deployment-8df5fbf9b-8m6f8:/# echo "<h1>Welcome CKA Students</h1>" > /usr/share/nginx/html/index.html
root@nginx-deployment-8df5fbf9b-8m6f8:/# curl localhost
<h1>Welcome CKA Students</h1>
```

We've made a change to the local filesystem of the Pod. Lets terminate this Pod and see what happens when we curl it again:

```bash
david@fedora:~/cka$ kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-8df5fbf9b-8m6f8   1/1     Running   0          32m

david@fedora:~/cka$ kubectl delete po nginx-deployment-8df5fbf9b-8m6f8
pod "nginx-deployment-8df5fbf9b-8m6f8" deleted from default namespace

david@fedora:~/cka$ kubectl get po
NAME                               READY   STATUS              RESTARTS   AGE
nginx-deployment-8df5fbf9b-hzgxc   0/1     ContainerCreating   0          2s
david@fedora:~/cka$ kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-8df5fbf9b-hzgxc   1/1     Running   0          8s

david@fedora:~/cka$ kubectl exec -it nginx-deployment-8df5fbf9b-hzgxc -- bash
root@nginx-deployment-8df5fbf9b-hzgxc:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
....
```

Note how we get the default nginx welcome page.


This setup is represented in the diagram below. If the Pod terminates, so does its ephemeral storage.

``` mermaid
graph TD
    subgraph pod [Kubernetes Pod]
        container(nginx container)
        storage[(ephemeral storage)]

        container link1@-->|"read/write"| storage
        
        link1@{ animate: true }
    end

    style pod rx:10,ry:10
```

!!! note "Note"

    Not all workloads require storage. Stateless workloads such as web frontends are a good example of this. Stateful workload examples include services such as databases and message queues that, at least in production, will definitely need persistant storage. 

By leveraging storage mechanisms in K8s we can decouple the application data from the Pod itself:

``` mermaid
%%{init: {'themeVariables': {'edgeLabelBackground': 'transparent'}}}%%
graph TD
    subgraph pod [Kubernetes Pod]
        container(nginx container)
    end

    pvc[(PVC: my-nginx-data)]

    container link1@-->|"read/write"| pvc
    
    link1@{ animate: true }

    style pod rx:10,ry:10
```

By doing this, the Data has its own lifecycle and operates independently to the Pod. We leverage the `PersistentVolumeClaim` API to achieve this (more on this in the next part)