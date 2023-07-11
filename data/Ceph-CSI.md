### Ceph-CSI 
#### Integrate your external ceph cluster with kubernetes. 

**To install and configure Ceph-CSI using Helm, you can follow these steps:**

  - Make sure you have Helm installed on your system. If not, you can install Helm by following the official Helm documentation: https://helm.sh/docs/intro/install/

  - Add the Ceph-CSI Helm repository by running the following command:

```
$ helm repo add ceph-csi https://ceph.github.io/csi-charts
```

```diff
$ helm repo list 
NAME            URL                                       
bitnami         https://charts.bitnami.com/bitnami        
ingress-nginx   https://kubernetes.github.io/ingress-nginx
+ceph-csi        https://ceph.github.io/csi-charts         
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

  - Update the Helm repositories by running:

```
helm repo update
```

  - Create a values.yaml file to customize the installation. Here's a sample values.yaml file that you can modify according to your environment:

```
kuser@saif-osa-bionic-deploy:~/ceph-csi$ cat values.yaml 
csiConfig:
  - clusterID: "b9db2ddd-41c0-4a67-8478-b40ee5b2de3c"
    monitors:
      - "192.168.122.91:6789"
provisioner:
  name: provisioner
  replicaCount: 1
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

  - In the above configuration, replace  <your-cluster-id> with your Kubernetes cluster ID  `` On ceph node execute # ceph mon dump``. Set the <monitors> to the desired monitor [mon] ip.

  - Verify that the Ceph-CSI chart is available by running:

```
$ helm search repo ceph-csi/ceph-csi
```

```diff
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ceph-csi/ceph-csi-cephfs        3.9.0           3.9.0           Container Storage Interface (CSI) driver, provi...
+ceph-csi/ceph-csi-rbd           3.9.0           3.9.0           Container Storage Interface (CSI) driver, provi...
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

  - Install Ceph-CSI using Helm, specifying the custom values.yaml file:

```diff
+$ helm install ceph-csi ceph-csi/ceph-csi-rbd --values values.yaml 
```

  - This command will install the Ceph-CSI chart with the provided configuration.

  - Verify the installation by checking the installed Helm release:

```
$ helm ls
```

```diff
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
+ceph-csi        default         1               2023-07-11 16:51:13.713176418 +0000 UTC deployed        ceph-csi-rbd-3.9.0      3.9.0      
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

  - Cross checking:- 

```
$ kubectl get po 
NAME                                                 READY   STATUS              RESTARTS   AGE
ceph-csi-ceph-csi-rbd-nodeplugin-lzgvp               3/3     Running             0          27m
ceph-csi-ceph-csi-rbd-provisioner-57674586db-ngtnc   7/7     Running             0          27m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ kubectl get all
NAME                                                     READY   STATUS              RESTARTS   AGE
pod/ceph-csi-ceph-csi-rbd-nodeplugin-lzgvp               3/3     Running             0          29m
pod/ceph-csi-ceph-csi-rbd-provisioner-57674586db-ngtnc   7/7     Running             0          29m
pod/ceph-rbd-pod-pvc-sc                                  0/1     ContainerCreating   0          24m

NAME                                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/ceph-csi-ceph-csi-rbd-nodeplugin-http-metrics    ClusterIP   10.97.110.88    <none>        8080/TCP   29m
service/ceph-csi-ceph-csi-rbd-provisioner-http-metrics   ClusterIP   10.98.158.217   <none>        8080/TCP   29m
service/kubernetes                                       ClusterIP   10.96.0.1       <none>        443/TCP    6h57m

NAME                                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/ceph-csi-ceph-csi-rbd-nodeplugin   1         1         1       1            1           <none>          29m

NAME                                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ceph-csi-ceph-csi-rbd-provisioner   1/1     1            1           29m

NAME                                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/ceph-csi-ceph-csi-rbd-provisioner-57674586db   1         1         1       29m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ kubectl get cm
NAME                             DATA   AGE
ceph-config                      2      29m
ceph-csi-config                  2      29m
ceph-csi-encryption-kms-config   1      29m
kube-root-ca.crt                 1      6h57m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```diff
$ kubectl  describe cm ceph-csi-config
Name:         ceph-csi-config
Namespace:    default
Labels:       app=ceph-csi-rbd
              app.kubernetes.io/managed-by=Helm
              chart=ceph-csi-rbd-3.9.0
              component=nodeplugin
              heritage=Helm
              release=ceph-csi
Annotations:  meta.helm.sh/release-name: ceph-csi
              meta.helm.sh/release-namespace: default

Data
====
cluster-mapping.json:
----
[]
config.json:
----
+[{"clusterID":"b9db2ddd-41c0-4a67-8478-b40ee5b2de3c","monitors":["192.168.122.91:6789"]}]

BinaryData
====

Events:  <none>
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```



  - You should see the Ceph-CSI release listed.
  - Congratulations! You have successfully installed and configured Ceph-CSI using Helm. You can now use Ceph-CSI for persistent storage in your Kubernetes environment. Remember to adapt the values.yaml file to your specific needs and refer to the Ceph-CSI documentation for further configuration options and usage guidelines.

  - Secret creation:- 

```
$ cat Secret.yaml 
```

```diff
apiVersion: v1
kind: Secret
metadata:
-  name: csi-rbd-secret
  namespace: default
+stringData:
+  userID: admin
+  userKey: AQDfx2hjkoohBxAAYsFzT3DvAAt4XYqMrDpUDA==

kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ cat StorageClass.yaml 
```

```diff
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-rbd-sc
provisioner: rbd.csi.ceph.com
parameters:
-   clusterID: b9db2ddd-41c0-4a67-8478-b40ee5b2de3c
-   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
-   csi.storage.k8s.io/provisioner-secret-namespace: default
-   csi.storage.k8s.io/controller-expand-secret-name: csi-rbd-secret
-   csi.storage.k8s.io/controller-expand-secret-namespace: default
-   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
-   csi.storage.k8s.io/node-stage-secret-namespace: default  
-   pool: metrics
   imageFeatures: layering
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
   - discard
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ cat PersistentVolumeClaim.yaml 
```

```diff
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
-  name: ceph-rbd-sc-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
-  storageClassName: ceph-rbd-sc
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ kubectl  get secrets 
```

```diff
NAME                             TYPE                 DATA   AGE
- csi-rbd-secret                   Opaque               2      32m
sh.helm.release.v1.ceph-csi.v1   helm.sh/release.v1   1      33m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ kubectl get storageclasses.storage.k8s.io 
```

```diff
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
+ ceph-rbd-sc          rbd.csi.ceph.com           Delete          Immediate           true                   32m
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  7h2m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ kubectl get persistentvolumeclaims 
```

```diff
NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
+ceph-rbd-sc-pvc   Bound    pvc-614e44d0-0454-4087-a155-e2e27b9cdd20   2Gi        RWO            ceph-rbd-sc    32m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```

```
$ kubectl get pv
```

```diff
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   REASON   AGE
+pvc-614e44d0-0454-4087-a155-e2e27b9cdd20   2Gi        RWO            Delete           Bound    default/ceph-rbd-sc-pvc   ceph-rbd-sc             32m
kuser@saif-osa-bionic-deploy:~/ceph-csi$ 
```




