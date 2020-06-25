* A daemonset can be used to run replicas of a pod on specific or all nodes in an kubernetes cluster.
* A DaemonSet ensures that all (or some) Nodes run a copy of a Pod.
* As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster,
* Deleting a DaemonSet will clean up the Pods it created.
* Some typical uses of a DaemonSet are:

    * running a cluster storage daemon on every node
    * running a logs collection daemon on every node
    * running a node monitoring daemon on every node

* When creating daemonsets, the nodeSelector field is used to indicate the nodes on which the daemonset should deploy replicas.
