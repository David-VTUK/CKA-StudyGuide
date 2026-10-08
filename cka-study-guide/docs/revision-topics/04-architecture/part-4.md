# Manage the lifecycle of Kubernetes clusters

## Perform a version upgrade on a Kubernetes cluster using Kubeadm

First, install kubeadm to a specific version. This will determine the k8s version that it deploys:

```shell
sudo apt-get update && sudo apt-get install -y kubeadm=1.36.5-00 kubelet=1.36.5-00 kubectl=1.36.5-00 && sudo apt-mark hold kubeadm
```

Stand up a k8s cluster

```shell
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Add CNI

```shell
https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

To upgrade the underlying k8s cluster, we need to upgrade kubeadm.

Update kubeadm

```shell
sudo apt-mark unhold kubeadm
sudo apt-get install --only-upgrade kubeadm
```

Next we `plan` the upgrade - this won't change our cluster but will display what changes can be made:

```shell
sudo kubeadm upgrade plan

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
kubelet     1 x v1.36.5   v1.37.1

Upgrade to the latest stable version:

COMPONENT                 CURRENT   AVAILABLE
kube-apiserver            v1.36.5   v1.37.1
kube-controller-manager   v1.36.5   v1.37.1
kube-scheduler            v1.36.5   v1.37.1
kube-proxy                v1.36.5   v1.37.1
CoreDNS                   1.14.2    1.14.6
etcd                      3.6.8-1   3.7.0-0

You can now apply the upgrade by executing the following command:

kubeadm upgrade apply v1.37.1
```

**Important note:** kubelet must be upgraded manually after this step.

Upgrade the cluster:

```shell
kubeadm upgrade apply v1.37.1
```

upgrade Kubelet:

```shell
sudo apt-get install --only-upgrade kubelet kubectl
```

## Node Maintenance (Cordon and Drain)

* **`kubectl cordon <node>`**: Marks node as unschedulable. Prevents new Pods from being scheduled; existing Pods remain running.
* **`kubectl drain <node>`**: Gracefully evicts all Pods from the node.
  * `--ignore-daemonsets`: Required to proceed if DaemonSet-managed Pods exist.
  * `--force`: Required to evict unmanaged (standalone) Pods.
* **`kubectl uncordon <node>`**: Returns the node to active, schedulable status.

### Adding Nodes

* **Generate join command**: Run `kubeadm token create --print-join-command` on the control plane.
* Execute the resulting `kubeadm join` command on the new node.

### Removing Nodes

1. **Drain**: `kubectl drain <node> --ignore-daemonsets --force`
2. **Delete**: `kubectl delete node <node>` (removes the object from the cluster).
3. **Reset**: Run `kubeadm reset` directly on the decommissioned node to wipe local Kubernetes state.


## Implement etcd backup and restore

### Backing up etcd

Take a snapshot of the DB, then store it in a safe location:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
```

Verify the backup:

```shell
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 2125d542 |   364069 |        770 |  3.8 MB    |
+----------+----------+------------+------------+
```

### Restore to etcd

To perform a restore:

```shell
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```
