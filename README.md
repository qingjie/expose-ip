# Exposing an External IP Address to Access an Application in a Cluster
```
qzhao-mbp:k8s qzhao$ cat load-balancer-example.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```
qzhao-mbp:k8s qzhao$ kubectl create -f load-balancer-example.yaml
deployment.apps/hello-world created

1. Display information about the Deployment:
```
qzhao-mbp:k8s qzhao$ kubectl get deployments hello-world
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   5/5     5            5           28m
qzhao-mbp:k8s qzhao$ kubectl describe deployments hello-world
Name:                   hello-world
Namespace:              default
CreationTimestamp:      Wed, 16 Oct 2019 09:40:50 -0400
Labels:                 app.kubernetes.io/name=load-balancer-example
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app.kubernetes.io/name=load-balancer-example
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app.kubernetes.io/name=load-balancer-example
  Containers:
   hello-world:
    Image:        gcr.io/google-samples/node-hello:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-world-bbbb4c85d (5/5 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  29m   deployment-controller  Scaled up replica set hello-world-bbbb4c85d to 5
```
2. Display information about your ReplicaSet objects:
```
qzhao-mbp:k8s qzhao$ kubectl get replicasets
NAME                    DESIRED   CURRENT   READY   AGE
hello-world-bbbb4c85d   5         5         5       29m
qzhao-mbp:k8s qzhao$ kubectl describe replicasets
Name:           hello-world-bbbb4c85d
Namespace:      default
Selector:       app.kubernetes.io/name=load-balancer-example,pod-template-hash=bbbb4c85d
Labels:         app.kubernetes.io/name=load-balancer-example
                pod-template-hash=bbbb4c85d
Annotations:    deployment.kubernetes.io/desired-replicas: 5
                deployment.kubernetes.io/max-replicas: 7
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/hello-world
Replicas:       5 current / 5 desired
Pods Status:    5 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app.kubernetes.io/name=load-balancer-example
           pod-template-hash=bbbb4c85d
  Containers:
   hello-world:
    Image:        gcr.io/google-samples/node-hello:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  29m   replicaset-controller  Created pod: hello-world-bbbb4c85d-vgcqq
  Normal  SuccessfulCreate  29m   replicaset-controller  Created pod: hello-world-bbbb4c85d-rpsmm
  Normal  SuccessfulCreate  29m   replicaset-controller  Created pod: hello-world-bbbb4c85d-fcdlf
  Normal  SuccessfulCreate  29m   replicaset-controller  Created pod: hello-world-bbbb4c85d-zfn2n
  Normal  SuccessfulCreate  29m   replicaset-controller  Created pod: hello-world-bbbb4c85d-jrwcq
```
3. Create a Service object that exposes the deployment:
```
qzhao-mbp:k8s qzhao$ kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
service/my-service exposed
```
4. Display information about the Service:
```
qzhao-mbp:k8s qzhao$ kubectl get services my-service
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
my-service   LoadBalancer   10.103.112.9   <pending>     8080:31177/TCP   15s
```

Note: If the external IP address is shown as <pending>, wait for a minute and enter the same command again.

5. Display detailed information about the Service:
```
qzhao-mbp:k8s qzhao$ kubectl describe services my-service
Name:                     my-service
Namespace:                default
Labels:                   app.kubernetes.io/name=load-balancer-example
Annotations:              <none>
Selector:                 app.kubernetes.io/name=load-balancer-example
Type:                     LoadBalancer
IP:                       10.103.112.9
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31177/TCP
Endpoints:                172.17.0.5:8080,172.17.0.6:8080,172.17.0.7:8080 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Make a note of the external IP address(LoadBalancer Ingress) exposed by your service. In this example, the external IP Address
is 192.168.99.104. Also note the value of ```Port``` and ```NodePort```. In this example, the ```Port``` is 8080 and the ```NodePort``` is 31177

6. In the preceding output, you can see that the service has several endpoints:
172.17.0.5:8080,172.17.0.6:8080,172.17.0.7:8080 + 2 more.These are internal addresses of the pods that are running the Hello World application. 
To verify these are pod addresses, enter
this command:
```
qzhao-mbp:k8s qzhao$ kubectl get pods --output=wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
hello-world-bbbb4c85d-fcdlf   1/1     Running   0          33m   172.17.0.7   minikube   <none>           <none>
hello-world-bbbb4c85d-jrwcq   1/1     Running   0          33m   172.17.0.6   minikube   <none>           <none>
hello-world-bbbb4c85d-rpsmm   1/1     Running   0          33m   172.17.0.5   minikube   <none>           <none>
hello-world-bbbb4c85d-vgcqq   1/1     Running   0          33m   172.17.0.8   minikube   <none>           <none>
hello-world-bbbb4c85d-zfn2n   1/1     Running   0          33m   172.17.0.9   minikube   <none>           <none>
```
7. Use the external IP address(LoadBalancer Ingress) to access the Hello World application:
```
curl http://<external-ip>:<port>
```
where <external-ip> is the external IP address (LoadBalancer Ingress) of your Service, and <port> is the value of Port in your Service description. 
The response to a successful request is a hello message:
```
Hello Kubernetes!
```

If you are using minikube, typing minikube service my-service will automatically open the Hello World application in a browser.
```
qzhao-mbp:k8s qzhao$ minikube service my-service
|-----------|------------|-------------|-----------------------------|
| NAMESPACE |    NAME    | TARGET PORT |             URL             |
|-----------|------------|-------------|-----------------------------|
| default   | my-service |             | http://192.168.99.104:31177 |
|-----------|------------|-------------|-----------------------------|
🎉  Opening kubernetes service  default/my-service in default browser...
```

# Cleaning up
To delete the Service, enter this command:
```
kubectl delete services my-service
```
To delete the Deployment, the ReplicaSet, and the Pods that are running the Hello World application, enter this command:
```
kubectl delete deployment hello-world
```


## Note:
How to get IP
```
qzhao-mbp:~ qzhao$ minikube ip
192.168.99.104
```
```
qzhao-mbp:~ qzhao$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl http://192.168.99.104:31177
Hello Kubernetes!$
```
