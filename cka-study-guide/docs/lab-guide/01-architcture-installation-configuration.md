# Lab Exercises for Cluster Architecture, Installation and Configuration

## Exercise 0 - Setup

* Prepare a cluster (Single node, kubeadm, k3s, etc)
* Prepare two vanilla VM's (No Kubernetes components installed) with the kubeadm binary installed (feel free to do this beforehand, I doubt it will be a requirement to add apt sources to get it)
* Open browser tabs to [Kubernetes Documentation](https://kubernetes.io/docs/), [Kubernetes GitHub](https://github.com/kubernetes/) and [Kubernetes Blog](https://kubernetes.io/blog/) (these are permitted as per [the current guidelines](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad))

## Exercise 1 - RBAC

A third party application requires access to describe `job` objects that reside in a namespace called `rbac`. Perform the following:

1. Create a namespace called `rbac`
2. Create a service account called `job-inspector` for the `rbac` namespace
3. Create a role that has rules to `get` and `list` job objects
4. Create a rolebinding that binds the service account `job-inspector` to the role created in step 3
5. Prove the `job-inspector` service account can "get" `job` objects but not `deployment` objects

??? "Answer - Imperative"

    ```shell
    kubectl create namespace rbac
    kubectl create sa job-inspector -n rbac
    kubectl create role job-inspector --verb=get --verb=list --resource=jobs -n rbac
    kubectl create rolebinding permit-job-inspector --role=job-inspector --serviceaccount=rbac:job-inspector -n rbac
    kubectl --as=system:serviceaccount:rbac:job-inspector auth can-i get job -n rbac 
    kubectl --as=system:serviceaccount:rbac:job-inspector auth can-i get deployment -n rbac
    ```

??? "Answer - Declarative"

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: rbac
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: job-inspector
      namespace: rbac
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: job-inspector
      namespace: rbac
    rules:
      - apiGroups: ["batch"]
        resources: ["jobs"]
        verbs: ["get", "list"]
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: permit-job-inspector
      namespace: rbac
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: job-inspector
    subjects:
      - kind: ServiceAccount
        name: job-inspector
        namespace: rbac
    ```

## Exercise 2 - Use Kubeadm to install a basic cluster

1. On a node, install kubeadm and stand up the control plane, using `10.244.0.0/16` as the pod network CIDR, and [kube-flannel.yml](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml) as the CNI
2. On a node, install kubeadm and join it to the cluster as a worker node

??? Answer

    Node 1:

    Prep kubeadm (as mentioned above, I doubt we will need to do this part in the exam)

    ```shell
    # Install a container runtime, IE https://github.com/containerd/containerd/blob/main/docs/getting-started.md
    apt-get update && apt-get install -y apt-transport-https curl
    curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    apt-get update
    apt-get install -y kubelet kubeadm kubectl
    apt-mark hold kubelet kubeadm kubectl
    ```

    Turn this node into a master

    ```shell
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ...
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ...
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ...
    Note the join command, ie:
    kubeadm join 172.16.10.210:6443 --token 9tjntl.10plpxqy85g8a0ui \
        --discovery-token-ca-cert-hash sha256:381165c9a9f19a123bd0fee36fe36d15e918062dcc94711ff5b286ee1f86b92b 
    ```
    Node 2:

    Run the join command taken from the previous step

    ```shell
    kubeadm join 172.16.10.210:6443 --token 9tjntl.10plpxqy85g8a0ui \
        --discovery-token-ca-cert-hash sha256:381165c9a9f19a123bd0fee36fe36d15e918062dcc94711ff5b286ee1f86b92b 
    ```

    Validate by running `kubectl get no` on the master node:

    ```shell
    kubectl get no
    NAME      STATUS   ROLES                  AGE     VERSION
    ubuntu    Ready    control-plane,master   9m53s   v1.26.0
    ubuntu2   Ready    <none>                 50s     v1.26.0
    ```

## Exercise 3 - Manage a highly-available Kubernetes cluster

1. Using `etcdctl`, determine the health of the etcd cluster
2. Using `etcdctl`, identify the list of members
3. On the master node, determine the health of the cluster by probing the API endpoint

??? Answer

    ```shell
    etcdctl cluster-health

    cluster is healthy
    member <id> is healthy
    member <id> is healthy
    member <id> is healthy

    etcdctl member list
    <id>: name=etcd1 peerURLs=http://<ip>:2380 clientURLs=<ip>:2379
    <id>: name=etcd0 peerURLs=http://<ip>:2380 clientURLs=<ip>:2379
    <id>: name=etcd2 peerURLs=http://<ip>:2380 clientURLs=<ip>:2379

    curl -k https://localhost:6443/healthz?verbose
    [+]ping ok
    [+]log ok
    [+]etcd ok
    [+]poststarthook/start-kube-apiserver-admission-initializer ok
    ...
    ```

## Exercise 4 - Perform a version upgrade on a Kubernetes cluster using Kubeadm

1. Using `kubeadm`, upgrade a cluster to the lastest version

??? Answer

    If held, unhold the kubeadm version

    ```shell
    sudo apt-mark unhold kubeadm
    ```

    Upgrade the `kubeadm` version:

    ```shell
    sudo apt-get install --only-upgrade kubeadm
    ```

    `plan` the upgrade:

    ```shell
    sudo kubeadm upgrade plan

    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    kubelet     1 x v1.19.0   v1.20.2

    Upgrade to the latest stable version:

    COMPONENT                 CURRENT   AVAILABLE
    kube-apiserver            v1.19.7   v1.20.2
    kube-controller-manager   v1.19.7   v1.20.2
    kube-scheduler            v1.19.7   v1.20.2
    kube-proxy                v1.19.7   v1.20.2
    CoreDNS                   1.7.0     1.7.0
    etcd                      3.4.9-1   3.4.13-0
    ```

    Upgrade the cluster

    ```shell
    kubeadm upgrade apply v1.20.2
    ```

    Upgrade Kubelet:

    ```shell
    sudo apt-get install --only-upgrade kubelet
    ```

## Exercise 5 - Implement etcd backup and restore

1. Take a backup of etcd
2. Verify the etcd backup has been successful
3. Restore the backup back to the cluster

??? Answer

    Take a snapshot of etcd:

    ```shell
    ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
    ```

    Verify the snapshot:

    ```shell
    sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
    ```

    Perform a restore:

    ```shell
    ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
    ```
