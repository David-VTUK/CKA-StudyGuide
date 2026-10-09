# Implement and configure a highly-available control plane

The previous section demonstrated creating a K8s cluster with one control plane node and several worker nodes - this does not provide resilience for the control plane. Several topologies exist for doing so:

## Stacked etcd

```mermaid
---
title: kubeadm HA topology - stacked etcd
---
graph TD
    %% Worker Nodes
    W1[worker node]
    W2[worker node]
    W3[worker node]
    W4[worker node]
    W5[worker node]

    %% Load Balancer
    LB[load balancer]

    %% Connections from workers to LB
    W1 -.-> LB
    W2 -.-> LB
    W3 -.-> LB
    W4 -.-> LB
    W5 -.-> LB

    %% Control Plane Node 1
    subgraph CP1 [control plane node]
        API1[apiserver]
        CM1[controller-manager]
        SCH1[scheduler]
        ETCD1[🛢️ etcd]
        
        API1 --- CM1
        CM1 --- SCH1
        SCH1 --- ETCD1
    end

    %% Control Plane Node 2
    subgraph CP2 [control plane node]
        API2[apiserver]
        CM2[controller-manager]
        SCH2[scheduler]
        ETCD2[🛢️ etcd]
        
        API2 --- CM2
        CM2 --- SCH2
        SCH2 --- ETCD2
    end

    %% Control Plane Node 3
    subgraph CP3 [control plane node]
        API3[apiserver]
        CM3[controller-manager]
        SCH3[scheduler]
        ETCD3[🛢️ etcd]
        
        API3 --- CM3
        CM3 --- SCH3
        SCH3 --- ETCD3
    end

    %% Connections from LB to API Servers
    LB -.-> API1
    LB -.-> API2
    LB -.-> API3

    %% API Server to ETCD loop connections
    ETCD1 <--> API1
    ETCD2 <--> API2
    ETCD3 <--> API3

    %% Stacked ETCD Cluster Indicator
    ETCD1 -. "stacked etcd cluster" .- ETCD2
    ETCD2 -. "stacked etcd cluster" .- ETCD3
```

* Multiple worker nodes
* Multiple control plane nodes fronted by a loadbalancer
* Embedded etcd within control plane

Notes:

etcd is quorum based. Therefore, if using stacked control plane nodes with etcd, odd numbers must be used.

### External etcd

```mermaid
---
title: kubeadm HA topology - external etcd
---
graph TD
    %% Worker Nodes
    W1[worker node]
    W2[worker node]
    W3[worker node]
    W4[worker node]
    W5[worker node]

    %% Load Balancer
    LB[load balancer]

    %% Connections from workers to LB (Solid lines)
    W1 --> LB
    W2 --> LB
    W3 --> LB
    W4 --> LB
    W5 --> LB

    %% Control Plane Node 1
    subgraph CP1 [control plane node]
        API1[apiserver]
        CM1[controller-manager]
        SCH1[scheduler]
        
        API1 --- CM1
        CM1 --- SCH1
    end

    %% Control Plane Node 2
    subgraph CP2 [control plane node]
        API2[apiserver]
        CM2[controller-manager]
        SCH2[scheduler]
        
        API2 --- CM2
        CM2 --- SCH2
    end

    %% Control Plane Node 3
    subgraph CP3 [control plane node]
        API3[apiserver]
        CM3[controller-manager]
        SCH3[scheduler]
        
        API3 --- CM3
        CM3 --- SCH3
    end

    %% Connections from LB to API Servers (Dashed lines)
    LB -.-> API1
    LB -.-> API2
    LB -.-> API3

    %% External ETCD Cluster
    subgraph EXT_ETCD [external etcd cluster]
        ETCD1[🛢️ etcd host]
        ETCD2[🛢️ etcd host]
        ETCD3[🛢️ etcd host]
    end

    %% FORCE LAYOUT: Invisible links push the ETCD cluster below the schedulers
    SCH1 ~~~ ETCD1
    SCH2 ~~~ ETCD2
    SCH3 ~~~ ETCD3

    %% ACTUAL CONNECTIONS: API Server to External ETCD (Bidirectional dashed lines)
    API1 <-.-> ETCD1
    API2 <-.-> ETCD2
    API3 <-.-> ETCD3
```

Notes:

* Multiple worker nodes
* Multiple control plane nodes fronted by a loadbalancer
* Etcd is external of the k8s cluster

Notes:

Advantage with this setup is etcd and the control plane can be scaled and managed independently of each other. This provides greater flexibility at the expense of operational complexity.

!!! success "Exam Tip"

    etcd is quorum based. Therefore, you need an odd number of nodes greater than one to achieve High Availability. It is common for smaller clusters to colocate the control plane and etcd roles.