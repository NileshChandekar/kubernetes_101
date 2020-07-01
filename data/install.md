# kubernetes_101
## Install Kubernetes Cluster using kubeadm

Follow this documentation to set up a Kubernetes cluster on __CentOS 7__ on KVM

This documentation guides you in setting up a cluster with ``1`` master node and ``3`` worker node.

## Assumptions

|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|kmaster.example.com|192.168.100.10|CentOS 7|2G|2|
|Worker|kworker1.example.com|192.168.100.15|CentOS 7|2G|1|
|Worker|kworker2.example.com|192.168.100.16|CentOS 7|2G|1|
|Worker|kworker3.example.com|192.168.100.17|CentOS 7|2G|1|


## Check required ports
|Protocol|Direction|PortRange|Purpose|
|----|----|----|----|----|----|
|TCP|Inbound|6443|KubernetesAPIserver|


###

|TCP|Inbound|2379-2380|etcd server client API|
|TCP|Inbound|10250|Kubelet API|
|TCP|Inbound|10251|kube-scheduler|




## On both Kmaster and Kworker
Perform all the commands as root user unless otherwise specified
### Pre-requisites
##### Update /etc/hosts
So that we can talk to each of the nodes in the cluster
```
cat >>/etc/hosts<<EOF
192.168.100.10 kmaster.example.com kmaster

192.168.100.15 kworker1.example.com kworker1
192.168.100.16 kworker2.example.com kworker2
192.168.100.17 kworker3.example.com kworker3

EOF
```
##### Install, enable and start docker service
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y  docker-ce
```

```
systemctl enable docker
systemctl start docker
```
##### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Disable Firewall
```
systemctl disable firewalld
systemctl stop firewalld
```
##### Disable swap
```
sed -i '/swap/d' /etc/fstab
swapoff -a
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Kubernetes Setup
##### Add yum repository
```
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
##### Install Kubernetes
```
yum install -y kubeadm kubelet kubectl
```
##### Enable and Start kubelet service
```
systemctl enable kubelet
systemctl start kubelet
```
## On kmaster
##### Initialize Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=192.168.100.10 --pod-network-cidr=172.18.100.0/24
```
##### Copy kube config
To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.

Lets create an reguler user account.

```
useradd cnilesh
echo cnilesh | passwd --stdin 0
echo "cnilesh ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/cnilesh
chmod 0440 /etc/sudoers.d/cnilesh
```

In my case, the user account is cnilesh

```
su - cnilesh
mkdir /home/cnilesh/.kube
sudo cp /etc/kubernetes/admin.conf /home/cnilesh/.kube/config
sudo chown -R cnilesh:cnilesh /home/cnilesh/.kube
```
##### Deploy Calico network
This has to be done as the user in the above step (in my case it is __cnilesh__)
```
kubectl create -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

##### Cluster join command
```
kubeadm token create --print-join-command
```
## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```
kubectl get nodes
```
##### Get component status
```
kubectl get cs
```

Have Fun!!


##### Check required ports [Master]

|Protocol|Direction|PortRange|Purpose|
|----|----|----|----|----|
|TCP|Inbound|6443*|Kubernetes API server|
|TCP|Inbound|2379-2380|etcd server client API|
|TCP|Inbound|10250|Kubelet API|
|TCP|Inbound|10251|kube-scheduler|
|TCP|Inbound|10252|kube-controller-manager|
|TCP|Inbound|10255|Read-only Kubelet API|

##### Check required ports [Worker]

|Protocol|Direction|Port Range|Purpose|
|----|----|----|----|----|
|TCP|Inbound|10250|Kubelet API|
|TCP|Inbound|10255|Read-only Kubelet API|
|TCP|Inbound|30000-32767|NodePort Services**|




### Day 2 ### Dashboard

##### Get Nodes status
```
[cnilesh@kmaster1 ~]$ kubectl get nodes
NAME                   STATUS   ROLES    AGE   VERSION
kmaster1.example.com   Ready    master   22h   v1.18.1
kworker1.example.com   Ready    <none>   22h   v1.18.1
kworker2.example.com   Ready    <none>   22h   v1.18.1
kworker3.example.com   Ready    <none>   22h   v1.18.1
[cnilesh@kmaster1 ~]$

```
##### Get component status
```
[cnilesh@kmaster1 ~]$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
[cnilesh@kmaster1 ~]$
```

##### Get component status
```
[cnilesh@kmaster1 ~]$ kubectl cluster-info
Kubernetes master is running at https://192.168.100.10:6443
KubeDNS is running at https://192.168.100.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
[cnilesh@kmaster1 ~]$
```

#### Get the version details.

```
[cnilesh@kmaster1 ~]$ kubectl version --short
Client Version: v1.18.1
Server Version: v1.18.1
[cnilesh@kmaster1 ~]$
```

### Before we procced, let me configure __kubectl__ on my local machine so I do not have to login into servers everytime.

~~~
https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux
~~~

~~~
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin/
sudo mv kubectl /usr/local/bin/
which kubectl
~~~

~~~
mkdir ~/.kube
scp root@192.168.100.10:/etc/kubernetes/admin.conf ~/.kube/config
~~~

* I am able to fire kubectl commands from my local system.

~~~
nchandek@cNilesh:~$ kubectl cluster-info
Kubernetes master is running at https://192.168.100.10:6443
KubeDNS is running at https://192.168.100.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
nchandek@cNilesh:~$
~~~

~~~
nchandek@cNilesh:~$ kubectl version --short
Client Version: v1.18.0
Server Version: v1.18.1
nchandek@cNilesh:~$
~~~

~~~
nchandek@cNilesh:~$ kubectl get nodes
NAME                   STATUS   ROLES    AGE   VERSION
kmaster1.example.com   Ready    master   23h   v1.18.1
kworker1.example.com   Ready    <none>   23h   v1.18.1
kworker2.example.com   Ready    <none>   23h   v1.18.1
kworker3.example.com   Ready    <none>   23h   v1.18.1
nchandek@cNilesh:~$
~~~

~~~
nchandek@cNilesh:~$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
nchandek@cNilesh:~$
~~~

* Get pod in namespaces: __kubectl -n kube-system get pods__

~~~
nchandek@cNilesh:~$ kubectl -n kube-system get pods
NAME                                           READY   STATUS    RESTARTS   AGE
calico-kube-controllers-75d56dfc47-xnw4v       1/1     Running   1          23h
calico-node-84bn9                              1/1     Running   1          23h
calico-node-9s988                              1/1     Running   1          23h
calico-node-mrdfs                              1/1     Running   1          23h
calico-node-rmm48                              1/1     Running   1          23h
coredns-66bff467f8-8hn77                       1/1     Running   1          23h
coredns-66bff467f8-r5744                       1/1     Running   1          23h
etcd-kmaster1.example.com                      1/1     Running   1          23h
kube-apiserver-kmaster1.example.com            1/1     Running   1          23h
kube-controller-manager-kmaster1.example.com   1/1     Running   1          23h
kube-proxy-6cmwb                               1/1     Running   1          23h
kube-proxy-ftj5d                               1/1     Running   1          23h
kube-proxy-hvmkt                               1/1     Running   1          23h
kube-proxy-n9488                               1/1     Running   1          23h
kube-scheduler-kmaster1.example.com            1/1     Running   1          23h
nchandek@cNilesh:~$
~~~

* The above output in more detail. __kubectl -n kube-system get pods -o wide__

~~~
NAME                                           READY   STATUS    RESTARTS   AGE    IP               NODE                   NOMINATED NODE   READINESS GATES
calico-kube-controllers-75d56dfc47-xnw4v       1/1     Running   2          2d8h   192.168.154.73   kmaster1.example.com   <none>           <none>
calico-node-84bn9                              1/1     Running   2          2d8h   192.168.100.16   kworker2.example.com   <none>           <none>
calico-node-9s988                              1/1     Running   2          2d8h   192.168.100.15   kworker1.example.com   <none>           <none>
calico-node-mrdfs                              1/1     Running   2          2d8h   192.168.100.10   kmaster1.example.com   <none>           <none>
calico-node-rmm48                              1/1     Running   2          2d8h   192.168.100.17   kworker3.example.com   <none>           <none>
coredns-66bff467f8-8hn77                       1/1     Running   2          2d8h   192.168.154.72   kmaster1.example.com   <none>           <none>
coredns-66bff467f8-r5744                       1/1     Running   2          2d8h   192.168.154.71   kmaster1.example.com   <none>           <none>
etcd-kmaster1.example.com                      1/1     Running   2          2d8h   192.168.100.10   kmaster1.example.com   <none>           <none>
kube-apiserver-kmaster1.example.com            1/1     Running   2          2d8h   192.168.100.10   kmaster1.example.com   <none>           <none>
kube-controller-manager-kmaster1.example.com   1/1     Running   2          2d8h   192.168.100.10   kmaster1.example.com   <none>           <none>
kube-proxy-6cmwb                               1/1     Running   2          2d8h   192.168.100.16   kworker2.example.com   <none>           <none>
kube-proxy-ftj5d                               1/1     Running   2          2d8h   192.168.100.15   kworker1.example.com   <none>           <none>
kube-proxy-hvmkt                               1/1     Running   2          2d8h   192.168.100.10   kmaster1.example.com   <none>           <none>
kube-proxy-n9488                               1/1     Running   2          2d8h   192.168.100.17   kworker3.example.com   <none>           <none>
kube-scheduler-kmaster1.example.com            1/1     Running   2          2d8h   192.168.100.10   kmaster1.example.com   <none>           <none>
=====
16
=====

~~~

### Now lets deploy dashboard.


```
kubectl apply  -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml
```

```
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$ kubectl apply  -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$
```

### Run kubectl proxy to log in to Dashboard.
__kubectl proxy &__

~~~
nchandek@cNilesh:~$ kubectl proxy &
[1] 14339
nchandek@cNilesh:~$ Starting to serve on 127.0.0.1:8001
~~~

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/1.png)

### To login into dashboard we have to create a service account. I have this below which I am going to deploy now.

~~~
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$ cat  k8s-dashboard-admin-user.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-systemnchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$
~~~

#### Lets apply it:
``kubectl apply -f k8s-dashboard-admin-user.yaml``

~~~
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$ kubectl apply -f k8s-dashboard-admin-user.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$
~~~

#### Lets check for service account which we have just created.

``kubectl get sa admin-user -n kube-system``
~~~
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$ kubectl get sa admin-user -n kube-system
NAME         SECRETS   AGE
admin-user   1         2m27s
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$
~~~

#### Lets check ``in detailed`` for service account which we have just created.
#### Make a note of Tokens filed.

``kubectl describe sa admin-user -n kube-system``

~~~
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$ kubectl describe sa admin-user -n kube-system
Name:                admin-user
Namespace:           kube-system
Labels:              <none>
Annotations:         Image pull secrets:  <none>
Mountable secrets:   admin-user-token-gndzp
Tokens:              admin-user-token-gndzp
Events:              <none>
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$
~~~

#### Lets check ``in detailed`` for Tokens, i.e. the secret tokens.
``kubectl describe  secret  admin-user-token-gndzp -n kube-system``
~~~
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$ kubectl describe  secret  admin-user-token-gndzp -n kube-system
Name:         admin-user-token-gndzp
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: dc76b1a4-1fd4-4bd2-8d5e-bea5f1e0665d

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImJDNVRWLUVXZnQtSnR6ZGVza2JNdnZJdmdueHJRRkQ2OG5pQ3RJVXBZdncifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWduZHpwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkYzc2YjFhNC0xZmQ0LTRiZDItOGQ1ZS1iZWE1ZjFlMDY2NWQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.OLUUtEgrqNp2MlhpxB4Et1RLK3oDXn24ewEFGQAiZk_y54253qxvjI7JBg5vknAIOMTgV3-HTfmK2aWVMVrI7UONpxEujFhSjJXhqYdpH7Fcb35TqjmsjbEo-oM4L0lBgxRuOk1Wn2swvzzyBWT3G8zjpFl2nVRaYGKDVCj5VybVj4ysu1QATgAc8gKiolkCJjRStnNK5IkKs_g9b4CYpA1z7a4ldXvX79Z5QivIz_rDPJifB372NUodTkpGYYICuxpM7Dr_d_VNLMG5svbDijqDr3Mq5bcOfSRMDPnzkK7a51Mznf8rC4RCs2FduaAPtkQVyk0GPab2X9ZqRrrFaw
nchandek@cNilesh:~/redhat/learning/kubernetes/dashboard$
~~~

#### Now use this token and try login into the Dashboard.

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/2.png)

#### The default overview of the Kubernetes Dashboard.
![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/3.png)

#### Cluster Nodes details in Dashboard.
![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/4.png)

#### Lets apply the new yaml ``matric-server``

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/5.png)

![Image ipa](https://github.com/NileshChandekar/kubernetes_101/blob/master/images/6.png)
