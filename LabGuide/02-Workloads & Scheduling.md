# Lab Exercises for Workloads & Scheduling

# Exercise 0 - Setup

* Prepare a cluster (Single node, kubedm, k3s, etc)
* Open browser tabs to https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad))

# Exercise 1 - Understand deployments and how to perform rolling update and rollbacks

1. Create a deployment object consisting of 6 pods, each containing a single `nginx:1.19.5` container
2. Identify the update strategy employed by this deployment
3. Modify the update strategy so `maxSurge` is equal to `50% `and `maxUnavailable` is equal to `50%`
4. Perform a rolling update to this deployment so that the image gets updated to `nginx1.19.6`
5. Undo the most recent change


<details><summary>Answer - Imperative</summary>

```shell
kubectl create deployment nginx --image=nginx:1.19.5 --replicas=6
kubectl describe deployment nginx (unless specified, will be listed as "UpdateStrategy")
kubectl patch deployment nginx -p "{\"spec\": {\"strategy\": {\"rollingUpdate\": { \"maxSurge\":\"50%\"}}}}" --record=true
kubectl patch deployment nginx -p "{\"spec\": {\"strategy\": {\"rollingUpdate\": { \"maxUnavailable\":\"50%\"}}}}" --record=true
kubectl set image deployment nginx nginx=nginx:1.19.6 --record=true
kubectl rollout undo deployment/nginx
```
</details>


<details><summary>Answer - Declarative</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 6
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.5
        ports:
        - containerPort: 80
```
```shell
kubectl describe deployment nginx (unless specified, will be listed as "UpdateStrategy")
```
`patch.yaml:`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
```
```shell
kubectl patch deployment nginx --patch-file=patch.yaml
```

```shell
kubectl rollout undo deployment/nginx
```
</details>

# Exercise 2 - Use ConfigMaps to configure applications

1. Create a configmap named `mycm` that has the following key=value pair
    1. `key` = owner
    2. `value` = yourname
2. Create a pod of your choice, such as `nginx`. Configure this Pod so that the underlying container has the environent varibale `OWNER` set to the value of this configmap 

<details><summary>Answer</summary>

Create Secret:
```shell
kubectl create configmap mycm --from-literal=owner=david
```

Define Pod:

```shell
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
spec:
  containers:
    - name: nginx-configmap
      image: nginx
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: OWNER
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: mycm
              # Specify the key associated with the value
              key: owner
```

Validate:

```shell
kubectl logs nginx-configmap | grep OWNER
OWNER=david
```
</details>

# Exercise 3 - Use Secrets to configure applications

<details><summary>Answer</summary>
1. Create a secret named `mysecret` that has the following key=value pair
    1. `dbuser` = MyDatabaseUser
    2. `dbpassword` = MyDatabasePassword
2. Create a pod of your choice, such as `nginx`. Configure this Pod so that the underlying container has the the following environment variables set:
  1. `DBUSER` from secret key `dbuser`
  2. `DBPASS` from secret key `dbpassword`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-web-secret
  labels:
    role: web
spec:
  containers:
  - name: nginx
    image: nginx
    env:
      - name: DBUSER
        valueFrom:
          secretKeyRef:
           name: db-credentials
           key: db-username
      - name: db_password
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: db-password
```

</details>