# Ingress Controller.

* Node Port

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/11112.png)

* Cluster-IP

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/11111.png)

* Requirement to have succesfull deployment of `INGRESS CONTROLLER`
* Resource
  * namespace
  * service account
  * cluster role
  * cluster role binding
  * config map
  * secret
  * DaemonSet
* Controller's:
  * NginX
  * Nginx+
  * Traefik  

* Practical # Ingress.

* Build a new VM for `HAPROXY`

```
[root@haproxy ~]# yum install haproxy -y
```

* `cat /etc/haproxy/haproxy.config`

```
frontend http_front
  bind *:80
  stats uri /haproxy?stats
  default_backend http_back

backend http_back
  balance roundrobin
  server kube 192.168.100.15:80
  server kube 192.168.100.16:80
  server kube 192.168.100.17:80
```

```
systemctl status haproxy
systemctl restart  haproxy
systemctl enable  haproxy
systemctl status haproxy
```

* Configure Ingress Controller.

* Check default namespaces // secrets

* `kubectl get ns`
```
NAME                   STATUS   AGE
default                Active   78d
kube-node-lease        Active   78d
kube-public            Active   78d
kube-system            Active   78d
kubernetes-dashboard   Active   75d
```

* `kubectl get secrets`
```
NAME                  TYPE                                  DATA   AGE
default-token-65jpj   kubernetes.io/service-account-token   3      78d
```
* `kubectl get clusterroles | grep -i nginx`  # no cluster role are defined yet.



* `git clone https://github.com/nginxinc/kubernetes-ingress.git`

  * Create a namespace and a service account for the Ingress controller:
  ```
  kubectl apply -f common/ns-and-sa.yaml
  ```  
  ```
  namespace/nginx-ingress created
  serviceaccount/nginx-ingress created
  ```
  * `kubectl get ns`

  ```diff
    NAME                   STATUS   AGE
    default                Active   78d
    kube-node-lease        Active   78d
    kube-public            Active   78d
    kube-system            Active   78d
    kubernetes-dashboard   Active   75d
  + nginx-ingress          Active   86s
  ```
  * Create a cluster role and cluster role binding for the service account:
  ```
  kubectl apply -f rbac/rbac.yaml
  ```
