# Prepare underlying infrastructure for installing a Kubernetes cluster

## Hardware and Identity Prerequisites

Before installing Kubernetes components, the raw machines (virtual or bare-metal) must meet baseline requirements to ensure the control plane and workers can function and communicate.

* **Minimum Resources:** At least 2 GB of RAM and 2 CPUs per machine.
* **Unique Identities:** Every node must have a unique Hostname, MAC address, and `product_uuid`. If these are duplicated (common in cloned VMs), the `kubelet` will fail to register the nodes properly.
* **Network Connectivity:** Full, unhindered network connectivity between all machines in the cluster (public or private network).

## Operating System and Kernel Configuration

Kubernetes requires specific kernel modules and network forwarding capabilities to route Pod traffic and manage network policies.

* **Disable Swap:** By default, the `kubelet` will fail to start if swap memory is enabled. You must disable it (e.g., `swapoff -a` and removing it from `/etc/fstab`). *(Note: While recent Kubernetes versions introduced beta support for swap, disabling it remains the standard requirement for traditional `kubeadm` installations).*
* **Load Kernel Modules:** The OS must load the `overlay` and `br_netfilter` modules to support the container runtime and network plugins.
* **Enable IPv4 Forwarding and iptables:** You must configure `sysctl` to allow the kernel to route traffic and ensure iptables can inspect bridged traffic. Required parameters include:
  * `net.ipv4.ip_forward = 1`
  * `net.bridge.bridge-nf-call-iptables = 1`
  * `net.bridge.bridge-nf-call-ip6tables = 1`

## Container Runtime Installation

Kubernetes no longer includes a default container runtime. You must install and configure a Container Runtime Interface (CRI) compatible runtime on every node before initializing the cluster.

* **Supported Runtimes:** Install `containerd` or `CRI-O`.
* **Cgroup Driver Configuration:** Both the container runtime and the Kubernetes `kubelet` must be configured to use the exact same cgroup driver (the software that limits and isolates resource usage). On Linux systems running `systemd`, you must configure your chosen container runtime to use the `systemd` cgroup driver rather than the default `cgroupfs` driver.

## Network and Firewall Port Configuration

If you are running a firewall (like `ufw` or `firewalld`), you must explicitly open the ports required for Kubernetes components to communicate.

* **Control Plane Nodes:**
  * `6443` (Kubernetes API Server)
  * `2379-2380` (etcd server client API)
  * `10250` (Kubelet API)
  * `10259` (kube-scheduler)
  * `10257` (kube-controller-manager)
* **Worker Nodes:**
  * `10250` (Kubelet API)
  * `30000-32767` (Default NodePort Services range)
* **Pod Network CIDR:** You must select and define a contiguous block of IP addresses (CIDR) that will be allocated to your Pods. This CIDR block must not overlap with the host network's IP range.
