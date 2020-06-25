### What Does “Kubernetes” Mean?
* Greek for “pilot” or
“Helmsman of a ship”
* Project that was spun out of Google as an open source
container orchestration platform.

### What can Kubernetes REALLY do?
* Autoscale Workloads
* Blue/Green Deployments
* Fire off jobs and scheduled cronjobs
* Manage Stateless and Stateful Applications
* Provide native methods of service discovery
* Easily integrate and support 3rd party apps
* Use the SAME API
across bare metal and
EVERY cloud provider!!!

### Architecture Overview

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/1113.png)

## Master Node or Control Plane Components

* kube-apiserver
* etcd
* kube-controller-manager
* kube-scheduler

##### kube-apiserver

  * Provides a forward facing REST interface into the kubernetes control plane and datastore.
  * All clients and other applications interact with kubernetes strictly through the API Server.
  * Acts as the gatekeeper to the cluster by handling
authentication and authorization, request validation,
mutation, and admission control in addition to being the
front-end to the backing datastore.

#### etcd

* etcd acts as the cluster datastore.
* Purpose in relation to Kubernetes is to provide a strong,
consistent and highly available key-value store for
persisting cluster state.
* Stores objects and config information.
It uses “Raft Consensus” alogorithum
among a quorum of systems
to create a fault-tolerant
consistent “view” of the
cluster.

#### kube-controller-manager

* Serves as the primary daemon that
manages all core component control loops.
* Monitors the cluster state via the apiserver
and steers the cluster towards the
desired state.

#### kube-scheduler

* Verbose policy-rich engine that evaluates workload
requirements and attempts to place it on a matching
resource.
* Default scheduler uses bin packing.
* Workload Requirements can include: general hardware
requirements, affinity/anti-affinity, labels, and other
various custom resource requirements.

## Worker Node Components

* kubelet
* kube-proxy
* Container Runtime Engine

#### kubelet

* Acts as the node agent responsible for managing the
lifecycle of every pod on its host.
* Kubelet understands YAML container manifests that it
can read from several sources:
  * file path
  * HTTP Endpoint
  * etcd watch acting on any changes
  * HTTP Server mode accepting container manifests
over a simple API.

#### kube-proxy

* Manages the network rules on each node.
* Performs connection forwarding or load balancing for
Kubernetes cluster services.
* Available Proxy Modes:
  * Userspace
  * iptables

#### Container Runtime Engine

* A container runtime is a CRI (Container Runtime
Interface) compatible application that executes and
manages containers.
  * Containerd (docker)
  * Cri-o
  * Rkt
  * kata (formerly clear and hyper)
  * Virtlet (VM CRI compatible runtime)


![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/1114.png)


## Optional Services

* cloud-controller-manager
* Cluster DNS
* Kube Dashboard
* Heapster / Metrics API Server
