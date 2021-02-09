# Understand host networking configuration on the cluster nodes

![img.png](images/networking.png)

At the host level, we have an interface (typically something like `eth0` or `ens192` etc) that acts as the primary network adapter.  

Each host is responsible for one subnet of the the CNI range. In this example, the left host is responsible for 10.1.1.0/24, and the right host 10.1.2.0/24.

Virtual ethernet adapters are paired with a corresponding Pod network adapter. Kernel routing is used to enable Pods to communicate outside of the host it resides in.

