# Create a simple POD.

* Currently no pods are running in the env [fresh setup]

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl get pod
No resources found in default namespace.
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ cat pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```

* Run the template [POD definition]

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl create -f pod-nginx.yaml
pod/nginx created
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl get pod
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          26s
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```
* Check whther created ot not.

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          105s
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```

* In detailed [long/wide] output

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP               NODE                   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m29s   192.168.136.74   kworker2.example.com   <none>           <none>
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```
* Where  
  * `nginx` = pod names
  * `192.168.136.74` = Container IP
  * `kworker2.example.com`    = node name where the pod is running.

# Create a simple POD from CLI.

```
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl run --generator=run-pod/v1 nginx1 --image=nginx
Flag --generator has been deprecated, has no effect and will be removed in the future.
pod/nginx1 created
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$
```

```diff
nchandek@cNilesh:~/redhat/learning/kubernetes/k8s_1_18$ kubectl get pod -o wide
NAME     READY   STATUS    RESTARTS   AGE     IP               NODE                   NOMINATED NODE   READINESS GATES
nginx    1/1     Running   0          90m     192.168.136.74   kworker2.example.com   <none>           <none>
+ nginx1   1/1     Running   0          5m51s   192.168.33.207   kworker1.example.com   <none>           <none>
```
