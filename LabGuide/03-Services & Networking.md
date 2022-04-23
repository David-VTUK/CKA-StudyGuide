# Lab Exercises for Services & Networking

# Exercise 0 - Setup

* Prepare a cluster (Single node, kubeadm, k3s, etc)
* Open browser tabs to https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-cerified-kubernetes-application-developer-ckad))

# Exercise 1 - Understand connectivity between Pods

1. Deploy [the following manifest](https://raw.githubusercontent.com/David-VTUK/CKAExampleYaml/master/nginx-svc-and-deployment.yaml)
2. Using `kubectl`, identify the Pod IP addresses
3. Determine the DNS name of the service.


<details><summary>Answer</summary>

Identify the `selector` for the service:

```shell
kubectl describe service nginx-service | grep -i selector
Selector:          app=nginx
```
Filter `kubectl` output:

```shell
kubectl get po -l app=nginx -o wide
```

Service name will be, based on the format `[Service Name].[Namespace].[Type].[Base Domain Name]` :

```shell
nginx-service.default.svc.cluster.local
```
</details>

# Exercise 2 - Understand ClusterIP, NodePort, LoadBalancer service types and endpoint

1. Create three `deployments` of your choosing
2. Expose one of these deployments with a service of type `ClusterIP`
3. Expose one of these deployments with a service of type `Nodeport`
4. Expose one of these deployments with a service of type `Loadbalancer`
    1. Note, this remains in `pending` status unless your cluster has integration with a cloud provider that provisions one for you (ie AWS ELB), or you have a software implementation such as `metallb`


<details><summary>Answer - Imperative</summary>

```shell
kubectl create deployment nginx-clusterip --image=nginx --replicas 1
kubectl create deployment nginx-nodeport --image=nginx --replicas 1
kubectl create deployment nginx-loadbalancer --image=nginx --replicas 1
```

```shell
kubectl expose deployment nginx-clusterip --type="ClusterIP" --port="80"
kubectl expose deployment nginx-nodeport --type="NodePort" --port="80"
kubectl expose deployment nginx-loadbalancer --type="LoadBalancer" --port="80"
```

</details>

<details><summary>Answer - Declarative</summary>

Apply the following:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx-clusterip
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-clusterip
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-clusterip
  template:
    metadata:
      labels:
        app: nginx-clusterip
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx-nodeport
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-nodeport
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-nodeport
  template:
    metadata:
      labels:
        app: nginx-nodeport
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: nginx-loadbalancer
spec:
  selector:
    app: nginx-loadbalancer
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-loadbalancer
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-loadbalancer
  template:
    metadata:
      labels:
        app: nginx-loadbalancer
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
</details>

# Exercise 3 - Know how to use Ingress controllers and Ingress resources

1. Create an `ingress` object named `myingress` with the following specification:

* Manages the host `myingress.mydomain`
* Traffic to the base path `/` will be forwarded to a `service` called `main` on port 80
* Traffic to the path `/api` will be forwarded to a `service` called `api` on port 8080


<details><summary>Answer - Imperative</summary>

```shell
kubectl create ingress myingress --rule="myingress.mydomain/=main:80" --rule="myingress.mydomain/api=api:8080"
```
</details>


<details><summary>Answer - Declarative</summary>

Apply the following YAML:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: myingress
spec:
  rules:
  - host: myingress.mydomain
    http:
      paths:
      - backend:
          service:
            name: main
            port:
              number: 80
        path: /
        pathType: Exact
      - backend:
          service:
            name: api
            port:
              number: 8080
        path: /api
        pathType: Exact
status:
  loadBalancer: {}
```

</details>


# Exercise 4 - Know how to configure and use CoreDNS

1. Identify the configuration location of `coredns`
2. Modify the coredns config file so DNS queries not resolved by itself are forwarded to the DNS server `8.8.8.8`
3. Validate the changes you have made
4. Add additional configuration so that all DNS queries for `custom.local` are forwarded to the resolver `10.5.4.223`


<details><summary>Answer</summary>

```shell
kubectl get cm coredns -n kube-system                                                
NAME      DATA   AGE
coredns   2      94d
```

```shell
kubectl edit cm coredns -n kube-system 

replace:
forward . /etc/resolv.conf

with
forward . 8.8.8.8
```

Add the block:
```shell
custom.local:53 {
        errors 
        cache 30
        forward . 10.5.4.223
        reload
    }
```

</details>