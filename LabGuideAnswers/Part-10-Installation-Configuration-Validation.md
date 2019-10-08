## CKA Lab Part 10 - Installation, Configuration & Validation

**Lab 1 - Master bringup using kubeadm**

* Create a master node VM

This is largely dependant on your infrastructure / lab. Three simple Ubuntu VMs will suffice in AWS/Azure/GCP/vSphere/etc

* Satisfy the prequisites

Ensure a container runtime is installed prior to installing kubeadm. For example, Docker instructions can be found at https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/

* Install the kubeadm and associated binaries

~~~
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubect
~~~

* Initialise the master assmuming you will leverage the `flannel` CNI

Note from https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

~~~
For flannel to work correctly, you must pass --pod-network-cidr=10.244.0.0/16 to kubeadm init.
~~~

Therefore, initalise accordingly:
~~~
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
~~~


* Apply the `flannel` CNI

~~~
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
~~~

**Lab 2 - Worker node bringup using kubeadm**

* Create a worker node VM

This is largely dependant on your infrastructure / lab. Three simple Ubuntu VM's will suffice in AWS/Azure/GCP/vSphere/etc

* Satisfy the prerequisites

Ensure a container runtime is installed prior to installing kubeadm. For example, Docker instructions can be found at https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/

* Install the kubeadm and associated binaries

~~~
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubect
~~~

* Join the worker to the cluster

~~~
kubeadm join [master][token information] - gathered from kubeadm init on master
~~~

* Validate the worker has joined the cluster

~~~
kubectl get nodes -o wide
~~~

**Lab 3 - Manage cluster wit Kubeadm**

* Generate a new token to add additional worker nodes the cluster

~~~
kubeadm token create --print-join-command
~~~
