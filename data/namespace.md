## Namespaces

* Logical boundary that prevent the overlap of configuration between the object and other set of configuration.
* Example - (a) Home dir in linux system, (b) creating the pod with the same name to avoid this use namespaces .
* Help different projects, teams, or customers to share a Kubernetes cluster.
* By default, a Kubernetes cluster will instantiate a default namespace when provisioning the cluster to hold the default set of Pods, Services, and Deployments used by the cluster.

* Namespaces are a logical cluster or environment, and are
the primary method of partitioning a cluster or scoping
access.


* Get the available namespaces by doing the following:

```
kubectl get namespaces
```

* Let's create two new namespaces to hold our work.

```
admin/namespace-dev.json
```
```
apiVersion: v1
kind: Namespace
metadata:
name: dev
labels:
app: MyBigWebApp_1
```

```
kubectl create -f https://k8s.io/examples/admin/namespace-dev.json
```


```
admin/namespace-prod.json
```
```
apiVersion: v1
kind: Namespace
metadata:
name: prod
labels:
app: MyBigWebApp_2
```
```
kubectl create -f https://k8s.io/examples/admin/namespace-prod.json
```


[WIP]

AI on me - get the output from the env.


## Default Namespaces

`default:`  The default namespace for any object without a namespace.
`kube-system:` Acts as the home for objects and resources created by Kubernetes itself.
