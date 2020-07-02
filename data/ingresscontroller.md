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

* `kubectl -n kube-system get pods | grep -i nginx` # no rbac defined yet.



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

  * Create a secret

  * `kubectl create -f common/default-server-secret.yaml`
  ```
  secret/default-server-secret created
  ```
  * `kubectl get secrets --all-namespaces | grep -i nginx`
  ```
  nginx-ingress          default-server-secret                            Opaque                                2      2m9s
  nginx-ingress          default-token-5gkmj                              kubernetes.io/service-account-token   3      9m23s
  nginx-ingress          nginx-ingress-token-4mjw2                        kubernetes.io/service-account-token   3      9m23s
  ```

  * Create a config map for customizing NGINX configuration:

  * `kubectl create -f common/nginx-config.yaml `
  ```
  configmap/nginx-config created
  ```
  * `kubectl get configmaps --all-namespaces`

    ```diff
      NAMESPACE              NAME                                 DATA   AGE
      kube-public            cluster-info                         1      78d
      kube-system            calico-config                        4      78d
      kube-system            coredns                              1      78d
      kube-system            extension-apiserver-authentication   6      78d
      kube-system            kube-proxy                           2      78d
      kube-system            kubeadm-config                       2      78d
      kube-system            kubelet-config-1.18                  1      78d
      kube-system            kubernetes-dashboard-settings        1      77d
      kubernetes-dashboard   kubernetes-dashboard-settings        0      75d
    + nginx-ingress          nginx-config                         0      58s
    ```

  * Create a cluster role and cluster role binding for the service account:
  ```
  kubectl create -f rbac/rbac.yaml
  ```
  * `kubectl create -f rbac/rbac.yaml`
  ```
  clusterrole.rbac.authorization.k8s.io/nginx-ingress created
  clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress created
  ```

  * Deploy the Ingress Controller
  * 2 ways to deploy Ingress controller:
    * `Deployment`
    * `DaemonSet`

# Using `DaemonSet`    

  * `kubectl create -f daemon-set/nginx-ingress.yaml`
  ```
  daemonset.apps/nginx-ingress created
  ```
  * `kubectl get daemonsets.apps -n nginx-ingress`
  ```
  NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
  nginx-ingress   3         3         3       3            3           <none>          102s
  ```
  * `kubectl get all -n nginx-ingress`
  ```
  NAME                      READY   STATUS    RESTARTS   AGE
  pod/nginx-ingress-2zzlk   1/1     Running   0          2m46s
  pod/nginx-ingress-588jf   1/1     Running   0          2m46s
  pod/nginx-ingress-l24tn   1/1     Running   0          2m46s

  NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
  daemonset.apps/nginx-ingress   3         3         3       3            3           <none>          2m47s
  ```

* Now the ingress controller is ready.

## Create pod
  * `kubectl create -f nginx-deploy-main.yaml`
  ```
  deployment.apps/nginx-deploy-main created
  ```

  * `kubectl get pods`
  ```
  NAME                                 READY   STATUS    RESTARTS   AGE
  nginx-deploy-main-545f4f6967-9hjkj   1/1     Running   0          52s
  nginx-deploy-main-545f4f6967-dgxb9   1/1     Running   0          52s
  nginx-deploy-main-545f4f6967-hjzwp   1/1     Running   0          52s
  ```

  * `for i in $(kubectl get pod | awk {'print $1'} | awk 'NR > 1'); do kubectl describe pod $i | grep -i node ; done`

    ```diff
    + Node:         kworker1.example.com/192.168.100.15
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    + Node:         kworker3.example.com/192.168.100.17
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    + Node:         kworker2.example.com/192.168.100.16
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    ```

  * `kubectl get all`

    ```
    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/nginx-deploy-main-545f4f6967-9hjkj   1/1     Running   0          10m
    pod/nginx-deploy-main-545f4f6967-dgxb9   1/1     Running   0          10m
    pod/nginx-deploy-main-545f4f6967-hjzwp   1/1     Running   0          10m

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d23h

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-deploy-main   3/3     3            3           10m

    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-deploy-main-545f4f6967   3         3         3       10m
    ```

## Create a Service or expose a service.

  *  `kubectl expose deployment  nginx-deploy-main --port 80`

    ```
    service/nginx-deploy-main exposed
    ```

  * `kubectl get all`

    ```diff
    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/nginx-deploy-main-545f4f6967-9hjkj   1/1     Running   0          15m
    pod/nginx-deploy-main-545f4f6967-dgxb9   1/1     Running   0          15m
    pod/nginx-deploy-main-545f4f6967-hjzwp   1/1     Running   0          15m

    NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP   4d23h
    + service/nginx-deploy-main   ClusterIP   10.105.165.88   <none>        80/TCP    61s

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-deploy-main   3/3     3            3           15m

    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-deploy-main-545f4f6967   3         3         3       15m
    ```

  * `kubectl get service`

    ```diff
    NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP   5d
    + nginx-deploy-main   ClusterIP   10.105.165.88   <none>        80/TCP    4m19s
    ```
  * `kubectl describe service nginx-deploy-main

    ```diff
    Name:              nginx-deploy-main
    Namespace:         default
    + Labels:            run=nginx
    Annotations:       <none>
    + Selector:          run=nginx-main
    + Type:              ClusterIP
    IP:                10.105.165.88
    Port:              <unset>  80/TCP
    TargetPort:        80/TCP
    + Endpoints:         192.168.136.81:80,192.168.201.210:80,192.168.33.214:80
    Session Affinity:  None
    Events:            <none>
    ```  

## Create Ingress Resource Rules:

* `kubectl create -f ingress-resource-1.yaml`
```
ingress.networking.k8s.io/ingress-resource-1 created
```

* `kubectl get ingress`
```
NAME                 CLASS    HOSTS                ADDRESS   PORTS   AGE
ingress-resource-1   <none>   nilesh.example.com             80      44s
```

* `kubectl describe ingress ingress-resource-1`

  ```diff
  + Name:             ingress-resource-1
  Namespace:        default
  Address:          
  Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
  Rules:
    Host                Path  Backends
    ----                ----  --------
  +   nilesh.example.com  
  +                          nginx-deploy-main:80  (192.168.136.81:80,192.168.201.210:80,192.168.33.214:80)
  Annotations:          <none>
  Events:
    Type    Reason          Age   From                      Message
    ----    ------          ----  ----                      -------
    Normal  AddedOrUpdated  90s   nginx-ingress-controller  Configuration for default/ingress-resource-1 was added or updated
    Normal  AddedOrUpdated  90s   nginx-ingress-controller  Configuration for default/ingress-resource-1 was added or updated
    Normal  AddedOrUpdated  90s   nginx-ingress-controller  Configuration for default/ingress-resource-1 was added or updated
  ```

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/11113.png)

* Lets delete the Ingress Rule,

* `kubectl delete ingress ingress-resource-1`
```
ingress.extensions "ingress-resource-1" deleted
```

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/11114.png)


* Lets try something else,

* `kubectl create -f nginx-deploy-firoz.yaml`

  ```
  deployment.apps/nginx-deploy-firoz created
  ```

* `kubectl create -f nginx-deploy-mukesh.yaml`

  ```
  deployment.apps/nginx-deploy-mukesh created
  ```

* `kubectl get all`

    ```diff
    NAME                                       READY   STATUS    RESTARTS   AGE
    pod/nginx-deploy-firoz-b9d974f8d-lzhfs     1/1     Running   0          2m57s
    pod/nginx-deploy-main-545f4f6967-9hjkj     1/1     Running   0          56m
    pod/nginx-deploy-main-545f4f6967-dgxb9     1/1     Running   0          56m
    pod/nginx-deploy-main-545f4f6967-hjzwp     1/1     Running   0          56m
    pod/nginx-deploy-mukesh-5958d6d547-pmffp   1/1     Running   0          2m6s

    NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP   5d
    service/nginx-deploy-main   ClusterIP   10.105.165.88   <none>        80/TCP    42m

    NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-deploy-firoz    1/1     1            1           2m57s
    deployment.apps/nginx-deploy-main     3/3     3            3           56m
    deployment.apps/nginx-deploy-mukesh   1/1     1            1           2m6s

    NAME                                             DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-deploy-firoz-b9d974f8d     1         1         1       2m57s
    replicaset.apps/nginx-deploy-main-545f4f6967     3         3         3       56m
    replicaset.apps/nginx-deploy-mukesh-5958d6d547   1         1         1       2m6s
    ```

* `kubectl expose deployment  nginx-deploy-firoz --port 80`

  ```
  service/nginx-deploy-firoz exposed
  ```

* `kubectl expose deployment  nginx-deploy-mukesh --port 80`

  ```
  service/nginx-deploy-mukesh exposed
  ```

* `kubectl get all`

    ```diff
    NAME                                       READY   STATUS    RESTARTS   AGE
    pod/nginx-deploy-firoz-b9d974f8d-lzhfs     1/1     Running   0          4m42s
    pod/nginx-deploy-main-545f4f6967-9hjkj     1/1     Running   0          58m
    pod/nginx-deploy-main-545f4f6967-dgxb9     1/1     Running   0          58m
    pod/nginx-deploy-main-545f4f6967-hjzwp     1/1     Running   0          58m
    pod/nginx-deploy-mukesh-5958d6d547-pmffp   1/1     Running   0          3m51s

    NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP   5d
    + service/nginx-deploy-firoz    ClusterIP   10.98.114.219    <none>        80/TCP    87s
    service/nginx-deploy-main     ClusterIP   10.105.165.88    <none>        80/TCP    44m
    + service/nginx-deploy-mukesh   ClusterIP   10.100.155.139   <none>        80/TCP    82s

    NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-deploy-firoz    1/1     1            1           4m42s
    deployment.apps/nginx-deploy-main     3/3     3            3           58m
    deployment.apps/nginx-deploy-mukesh   1/1     1            1           3m51s

    NAME                                             DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-deploy-firoz-b9d974f8d     1         1         1       4m42s
    replicaset.apps/nginx-deploy-main-545f4f6967     3         3         3       58m
    replicaset.apps/nginx-deploy-mukesh-5958d6d547   1         1         1       3m51s
    ```

* `kubectl create -f ingress-resource-2.yaml`

    ```
    ingress.extensions/ingress-resource-2 created
    ```

* `kubectl get ingress`

    ```
    NAME                 CLASS    HOSTS                                                    ADDRESS   PORTS   AGE
    ingress-resource-2   <none>   nginx.example.com,firoz.example.com,mukesh.example.com             80      37s
    ```

* `kubectl describe ingress ingress-resource-2`

    ```diff
    Name:             ingress-resource-2
    Namespace:        default
    Address:          
    Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
    Rules:
      Host                Path  Backends
      ----                ----  --------
    +   nginx.example.com   
                             nginx-deploy-main:80 (192.168.136.81:80,192.168.201.210:80,192.168.33.214:80)
    +   firoz.example.com   
                             nginx-deploy-firoz:80 (192.168.33.216:80)
    +   mukesh.example.com  
                             nginx-deploy-mukesh:80 (192.168.201.211:80)
    Annotations:          <none>
    Events:
      Type    Reason          Age   From                      Message
      ----    ------          ----  ----                      -------
      Normal  AddedOrUpdated  81s   nginx-ingress-controller  Configuration for default/ingress-resource-2 was added or updated
      Normal  AddedOrUpdated  81s   nginx-ingress-controller  Configuration for default/ingress-resource-2 was added or updated
      Normal  AddedOrUpdated  81s   nginx-ingress-controller  Configuration for default/ingress-resource-2 was added or updated
    ```

    ![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/11115.png)

    ![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/11116.png)
