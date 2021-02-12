# Lab Exercises for Workloads & Scheduling

# Exercise 0 - Setup

* Prepare a cluster (Single node, kubedm, k3s, etc)
* Open browser tabs to https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad))

# Exercise 1 - Understand deployments and how to perform rolling update and rollbacks

1. Create a deployment object consisting of 6 pods, each containing a single `nginx:1.19.5` container
2. Identify the update strategy employed by this deployment
3. Modify the update strategy so `maxSurge` is equal to `50% `and `maxUnavailable` is equal to `25%`
4. Perform a rolling update to this deployment so that the image gets updated to `nginx1.19.6`


<details><summary>Answer - Imperative</summary>

```shell
kubectl create deployment nginx --image=nginx:1.19.5 --replicas=6
kubectl describe deployment nginx (unless specified, will be listed as "UpdateStrategy")
```
</details>

kubectl patch deployment your_deployment -p "{\"spec\": {\"template\": {\"metadata\": { \"labels\": {  \"redeploy\": \"$(date +%s)\"}}}}}"
