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

* `kubectl create -f pod-nginx.yaml`

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


* get the yaml out of POD.

* `kubectl get pod nginx -o yaml`

```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.136.74/32
    cni.projectcalico.org/podIPs: 192.168.136.74/32
  creationTimestamp: "2020-07-01T10:38:53Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:containers:
          k:{"name":"nginx"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:ports:
              .: {}
              k:{"containerPort":80,"protocol":"TCP"}:
                .: {}
                f:containerPort: {}
                f:protocol: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
    manager: kubectl
    operation: Update
    time: "2020-07-01T10:38:53Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:cni.projectcalico.org/podIP: {}
          f:cni.projectcalico.org/podIPs: {}
    manager: calico
    operation: Update
    time: "2020-07-01T10:38:54Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"192.168.136.74"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2020-07-01T10:40:14Z"
  name: nginx
  namespace: default
  resourceVersion: "121779"
  selfLink: /api/v1/namespaces/default/pods/nginx
  uid: 0a6e2925-3f1e-4e9b-b6b5-d71edf6acfbf
spec:
  containers:
  - image: nginx:1.7.9
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-65jpj
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kworker2.example.com
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-65jpj
    secret:
      defaultMode: 420
      secretName: default-token-65jpj
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-07-01T10:38:53Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-07-01T10:40:14Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-07-01T10:40:14Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-07-01T10:38:53Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://f997384c4e4f895328f81165b5f035d284a71adfd7fb1c1ec8eda8f7ff214877
    image: nginx:1.7.9
    imageID: docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2020-07-01T10:40:14Z"
  hostIP: 192.168.100.16
  phase: Running
  podIP: 192.168.136.74
  podIPs:
  - ip: 192.168.136.74
  qosClass: BestEffort
  startTime: "2020-07-01T10:38:53Z"
```
