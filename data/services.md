* It is like a gateway that group all pods and provide a common ip to all the pods, so that we dont have to remember all ip address of the pods.
* So whenever we want to access any of the pods. we will simply call out service [IP+PORT]
* Service [IP+PORT] is directly map to PODS.
* Can be create based on the LABEL and SELECTOR.
* Service has,
  * cluster-ip
  * external-ip

* Unified method of accessing
  the exposed workloads of Pods.
* Durable resource
  * static cluster IP
  * static namespaced
  * DNS name  

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/1112.png)
