# Crash Course Kubernetes

## mongo secrets
```shell
echo -n mongouser | base64
echo -n mongopassword | base64
```
## procedure

### Start minikube
```shell
$ minikube start
```

### Giving Kubernetes the config files

* Below shows there is no components deployed yet
```shell
$ kubectl get pod
No resources found in default namespace.

```

* ConfigMap and Secret must exist before Deployments
```shell
$ kubectl apply -f .\demo\mongo-config.yaml
configmap/mongo-config created
$ kubectl apply -f .\demo\mongo-secret.yaml
secret/mongo-secret created

```

* Deploy mongo before webapp

```shell
$ kubectl apply -f .\demo\mongo.yaml
deployment.apps/mongo-deployment created
service/mongo-service created
```

* Last lets deploy the webapp
```shell
$ kubectl apply -f .\demo\webapp.yaml 
deployment.apps/webapp-deployment created
service/webapp-service created
```

### Interacting with the K8 Cluster
* To get all the components in the cluster run:
```shell
$ kubectl get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/mongo-deployment-77b4cbd6b-9flsx     1/1     Running   0          4m24s
pod/webapp-deployment-655ff6696b-4tvzl   1/1     Running   0          2m18s

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          22h
service/mongo-service    ClusterIP   10.100.187.140   <none>        27017/TCP        5m20s
service/webapp-service   NodePort    10.102.99.254    <none>        3000:30100/TCP   118s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo-deployment    1/1     1            1           4m24s
deployment.apps/webapp-deployment   1/1     1            1           2m18s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-deployment-77b4cbd6b     1         1         1       4m24s
replicaset.apps/webapp-deployment-655ff6696b   1         1         1       2m18s

```

**Note: Above does not show Secret or ConfigMap**

* `kubectl get pod | configmap | secret | ...`
```shell
$ kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      22h
mongo-config       1      10m
$ kubectl get secret   
NAME           TYPE     DATA   AGE
mongo-secret   Opaque   2      9m42s
```

* To get all the pods in the cluster

```shell
$ kubectl get pod   
NAME                                 READY   STATUS    RESTARTS   AGE
mongo-deployment-77b4cbd6b-9flsx     1/1     Running   0          8m26s
webapp-deployment-655ff6696b-4tvzl   1/1     Running   0          6m20s

```

### Get detailed information with `describe` command
* `kubectl describe <resourceType> <resourceName>`
* Lets try to get more details about webapp-service

```shell
$ kubectl describe service webapp-service
Name:                     webapp-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=webapp
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.102.99.254
IPs:                      10.102.99.254
Port:                     <unset>  3000/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  30100/TCP
Endpoints:                10.244.0.7:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```

* Lets get some more information about webapp pod

```shell
kubectl describe pod  webapp-deployment-655ff6696b-4tvzl
Name:             webapp-deployment-655ff6696b-4tvzl
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sun, 15 Dec 2024 01:22:53 -0800
Labels:           app=webapp
                  pod-template-hash=655ff6696b
Annotations:      <none>
Status:           Running
IP:               10.244.0.7
IPs:
  IP:           10.244.0.7
Controlled By:  ReplicaSet/webapp-deployment-655ff6696b
Containers:
  webapp:
    Container ID:   docker://8a6d45800759cef05aba27e4b26a05fce120f34a6de7369f5f2f0696d60f92fb
    Image:          nanajanashia/k8s-demo-app:v1.0
    Image ID:       docker-pullable://nanajanashia/k8s-demo-app@sha256:6f554135da39ac00a1c2f43e44c2b0b54ca13d3d8044da969361e7781adb7f95
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 15 Dec 2024 01:23:03 -0800
    Ready:          True
    Restart Count:  0
    Environment:
      USER_NAME:  <set to the key 'mongo-user' in secret 'mongo-secret'>      Optional: false
      USER_PWD:   <set to the key 'mongo-password' in secret 'mongo-secret'>  Optional: false
      DB_URL:     <set to the key 'mongo-url' of config map 'mongo-config'>   Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5rd9d (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-5rd9d:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  12m   default-scheduler  Successfully assigned default/webapp-deployment-655ff6696b-4tvzl to minikube
  Normal  Pulling    12m   kubelet            Pulling image "nanajanashia/k8s-demo-app:v1.0"
  Normal  Pulled     12m   kubelet            Successfully pulled image "nanajanashia/k8s-demo-app:v1.0" in 8.928s (8.928s including waiting). Image size: 124898678 bytes. 
  Normal  Created    12m   kubelet            Created container webapp
  Normal  Started    12m   kubelet            Started container webapp

```

### How to view logs of container
* `kubectl logs <podName>`

```shell
$ kubectl get pod                                         
NAME                                 READY   STATUS    RESTARTS   AGE
mongo-deployment-77b4cbd6b-9flsx     1/1     Running   0          16m
webapp-deployment-655ff6696b-4tvzl   1/1     Running   0          14m
$ kubectl logs webapp-deployment-655ff6696b-4tvzl 
app listening on port 3000!


```

### how to view all services
```shell
$ kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          23h
mongo-service    ClusterIP   10.100.187.140   <none>        27017/TCP        19m
webapp-service   NodePort    10.102.99.254    <none>        3000:30100/TCP   16m

```

* How do we access the external service `webapp-service` NodePort:
  * NodePort Service is accessible on each Worker Node's IP Address
  * In this case for `minikube we only have 1 node`

```shell
$ minikube ip
192.168.49.2
$ kubectl get node
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   32h   v1.31.0
$ kubectl get node -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                       CONTAINER-RUNTIME
minikube   Ready    control-plane   32h   v1.31.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   5.15.150.1-microsoft-standard-WSL2   docker://27.2.0


```