# Create a simple POD.

##### Currently no pods are running in the env [fresh setup]

* `kubectl get pod`

```
No resources found in default namespace.
```

* `cat pod-nginx.yaml`

```
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
```

* Run the template [POD definition]

* `kubectl create -f pod-nginx.yaml

```
pod/nginx created
```


* `kubectl get pod`

```
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          26s
```

* Check whther created ot not.


* `kubectl get pod`

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          105s
```

* In detailed [long/wide] output

* `kubectl get pod -o wide`

```
NAME    READY   STATUS    RESTARTS   AGE     IP               NODE                   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m29s   192.168.136.74   kworker2.example.com   <none>           <none>
```

* Where  
  * `nginx` = pod names
  * `192.168.136.74` = Container IP
  * `kworker2.example.com`    = node name where the pod is running.

# Create a simple POD from CLI.


* `kubectl run --generator=run-pod/v1 nginx1 --image=nginx`

```
pod/nginx1 created
```

* `date ; kubectl get pod -o wide`

```diff
  Wed Jul  1 17:41:06 IST 2020
  NAME     READY   STATUS    RESTARTS   AGE    IP               NODE                   NOMINATED NODE   READINESS GATES
  nginx    1/1     Running   0          92m    192.168.136.74   kworker2.example.com   <none>           <none>
+ nginx1   1/1     Running   0          8m4s   192.168.33.207   kworker1.example.com   <none>           <none>
```
