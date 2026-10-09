# Understand connectivity between Pods

Every Pod gets its own IP address on a flat network. This means you do not need to explicitly create links between Pods, and you almost never need to deal with mapping container ports to host ports. This creates a clean, backwards-compatible model where Pods can be treated much like VMs or physical hosts from the perspectives of port allocation, naming, service discovery, load balancing, application configuration and migration.

We can visualise this like so:

```mermaid
graph TD
    subgraph Node_A [Node A]
        direction LR
        subgraph Pod_A [Pod A]
            pA_eth0["eth0<br>10.1.1.1"]:::innerBox
        end
        class Pod_A pod;

        subgraph Pod_B [Pod B]
            pB_eth0["eth0<br>10.1.1.2"]:::innerBox
        end
        class Pod_B pod;

        veth123["veth123"]:::innerBox
        veth234["veth234"]:::innerBox
        bridgeA["Bridge<br>10.1.1.0/24"]:::bridge
        nA_eth0["eth0<br>10.100.0.1"]:::nodeEth

        pA_eth0 --- veth123
        pB_eth0 --- veth234
        veth123 --- bridgeA
        veth234 --- bridgeA
        bridgeA --- nA_eth0
    end

    subgraph Node_B [Node B]
        direction RL
        subgraph Pod_C [Pod C]
            pC_eth0["eth0<br>10.1.2.1"]:::innerBox
        end
        class Pod_C pod;

        subgraph Pod_D [Pod D]
            pD_eth0["eth0<br>10.1.2.2"]:::innerBox
        end
        class Pod_D pod;

        veth345["veth345"]:::innerBox
        veth456["veth456"]:::innerBox
        bridgeB["Bridge<br>10.1.2.0/24"]:::bridge
        nB_eth0["eth0<br>10.100.0.2"]:::nodeEth

        pC_eth0 --- veth345
        pD_eth0 --- veth456
        veth345 --- bridgeB
        veth456 --- bridgeB
        bridgeB --- nB_eth0
    end

    Network["Network"]:::networkBox

    nA_eth0 --- Network
    nB_eth0 --- Network
```

Kubernetes imposes the following fundamental requirements on any networking implementation (barring any intentional network segmentation policies):

* `Pods` on a node can communicate with all `Pods` on all nodes without NAT.
* Agents on a node (e.g. system daemons, Kubelet) can communicate with all pods on that node.

!!! success "Exam Tip"

    All `pods` inside a cluster operate as if they are on the same L2 network.

!!! success "Exam Tip"

    The `CNI` is responsible for a lot of the networking scaffolding a Pod is subject to.

!!! success "Exam Tip"

    Every `pod` gets an IP address, but `containers` within that Pod share it. Use port numbers to delimitate traffic between them.

!!! success "Exam Tip"

    By default, `pod` IP addresses are not accessible directly from outside the cluster. We use `services` to forward traffic to them from outside of the cluster.