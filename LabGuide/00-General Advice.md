# General Advice

# Imperative vs Declarative

The Lab guide, where feasible will presnt both imperative and declarative answers to address a question. Outside the exam, we should opreate in a Declarative manner. During the exam, however, time is an expensive commodity. Therefore, if you can, try and be as imperative as possible to save time.

# Use the documentation

At time of writing, https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ are permitted to be used during the exam as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad). If you need to get boiletplate YAML manifests to address an exam question, feel free to get them from here. The key is to know **what** to search for. 

## Caution on documentation

When using the Kubernetes.io docs, be careful of the search results - **some will point to resources outside of kubernetes.io and the list of accepted resource for the exam. Should you navigate to these your exam may be cancelled. Sanity check the URL the search result is pointing to prior to procedding.**

## Don't write YAML manifests from scratch

If you can, copy/paste from the docs. If you can't find what you're looking for, kubectl has a good way to generate YAML:

```shell
kubectl run nginx --image=nginx --dry-run -o yaml > pod.yaml
```

Which will generate:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

`--dry-run` won't commit the command  
`-o yaml` specifies YAML as the output  
`> pod.yaml` redirects the output to a file

At which point you can edit the object as you see fit before committing it to the cluster with `kubectl apply -f pod.yaml`. This can save a **lot** of time.

# Become BFF's with `kubectl`

Effective use of `kubectl` will be extremely important to pass the exam. Become very comfortable with it.

# Unsure of an objects spec, type, options?

`kubectl explain <resourcetype>`