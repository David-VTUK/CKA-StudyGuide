# Lab Exercises for Troubleshooting

# Exercise 0 - Setup

* Prepare a cluster (Single node, kubedm, k3s, etc)
* Open browser tabs to https://kubernetes.io/docs/, https://github.com/kubernetes/ and  https://kubernetes.io/blog/ (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-cerified-kubernetes-application-developer-ckad))
* Ensure your etcd nodes have `etcdctl` installed

# Exercise 1 - Evaluate cluster and node logging

1. For your cluster type, determine how to acquire logs for your master nodes. They could be in the form of:
    1. Services
    2. Static Pods
    3. Kubernetes Pods
2. Using `kubectl` get a list of `events` from your cluster
2. Using `etcdctl` determine the health of the etcd cluster

# Exercise 2 - Understand how to monitor applications



# Exercise 3 - Troubleshoot application failure

# Exercise 4 - Troubleshoot networking