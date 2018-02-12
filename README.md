- Deploy a grpc server

```console
kubectl apply -f https://raw.githubusercontent.com/appscode/hello-grpc/0.1.0/hack/deploy/deploy.yaml
```

- Deploy a websocket server

```console
kubectl apply -f https://raw.githubusercontent.com/appscode/hello-websocket/0.1.0/hack/deploy/deploy.yaml
```


```console
$ kubectl get pods,svc
NAME                                  READY     STATUS    RESTARTS   AGE
po/hello-grpc-fc5db84fc-5frs6         1/1       Running   0          13m
po/hello-grpc-fc5db84fc-mgl6v         1/1       Running   0          13m
po/hello-websocket-7bf77cf696-fsflq   1/1       Running   0          10m

NAME                  TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                                      AGE
svc/hello-grpc        LoadBalancer   10.15.240.84   35.224.2.46    80:30574/TCP,443:31396/TCP,56790:30768/TCP   13m
svc/hello-websocket   LoadBalancer   10.15.246.40   35.184.3.214   80:31716/TCP                                 10m
svc/kubernetes        ClusterIP      10.15.240.1    <none>         443/TCP                                      23m
```

Now, try the following URLs to confirm that grpc and websocket server is running:

![grpc-intro](/images/demo-grpc-intro.png)
![grpc-status](/images/demo-grpc-status.png)

![ws](/images/demo-ws.png)

- Install Voyager Operator
```console
curl -fsSL https://raw.githubusercontent.com/appscode/voyager/5.0.0-rc.11/hack/deploy/voyager.sh \
    | bash -s -- --provider=gke
```


- Now create ./deploy/ing.yaml

```console
$ kubectl apply -f ./deploy/ing.yaml
ingress "voyager-demo" created

$ kubectl get pods,svc
NAME                                       READY     STATUS    RESTARTS   AGE
po/hello-grpc-fc5db84fc-5frs6              1/1       Running   0          24m
po/hello-grpc-fc5db84fc-mgl6v              1/1       Running   0          24m
po/hello-websocket-7bf77cf696-fsflq        1/1       Running   0          21m
po/voyager-voyager-demo-67f458b5bc-bfskj   1/1       Running   0          53s

NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                      AGE
svc/hello-grpc             LoadBalancer   10.15.240.84    35.224.2.46      80:30574/TCP,443:31396/TCP,56790:30768/TCP   24m
svc/hello-websocket        LoadBalancer   10.15.246.40    35.184.3.214     80:31716/TCP                                 21m
svc/kubernetes             ClusterIP      10.15.240.1     <none>           443/TCP                                      34m
svc/voyager-voyager-demo   LoadBalancer   10.15.242.178   35.202.123.114   80:31374/TCP                                 53s
```


- Configure api.kiteci.com and ws.kiteci.com domains

![kiteci-domains](/images/kiteci-domains.png)


- Now wait to bit confirm that these new domains are resolving.

```console
$ dig +short api.kiteci.com
35.202.123.114

$ dig +short ws.kiteci.com
35.202.123.114
```

![api-domain](/images/api-domain.png)
![ws-domain](/images/ws-domain.png)

