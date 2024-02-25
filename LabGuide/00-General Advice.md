# General Advice

# Imperative vs Declarative

The Lab guide will present both imperative and declarative answers to questions where possible. Due to time constraints in the exam, using imperative commands may be more efficient. Outside of the exam, declarative management is the preferred approach.

# Use the documentation

At time of writing, https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ are permitted to be used during the exam as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad). If you need to get boiletplate YAML manifests to address an exam question, feel free to get them from here. The key is to know **what** to search for. 

For example, if asked to create and apply a `service` object, search for `Service YAML` in the Kubernetes docs and [modify an example](https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service) to suit the question objectives.

## Caution on documentation

When using the Kubernetes.io docs, be careful of the search results - **some will point to resources outside of kubernetes.io and the list of accepted resource for the exam. Should you navigate to these your exam may be cancelled. Sanity check the URL the search result is pointing to prior to procedding.**

## Don't write YAML manifests from scratch

In addition to the above example, leverage the combination of `--dry-run` and `-o` to generate YAML that can be modified. 

```shell
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
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

# Become BFF's with `kubectl`

Effective use of `kubectl` will be extremely important to pass the exam. Become very comfortable with it.

# Unsure of an objects spec, type, options?

`kubectl explain <resourcetype>`