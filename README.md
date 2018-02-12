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


- Create acme account secret

kubectl create secret generic acme-account --from-literal=ACME_EMAIL=tamal@appscode.com

- Now create the Certificate object. This will issue new TLS certs from Let's Encrypt and create a secret called tls-kiteci-com .

```console
$ kubectl apply -f ./deploy/certs.yaml
certificate "kiteci-com" created

$ kubectl get secrets
NAME                  TYPE                                  DATA      AGE
acme-account          Opaque                                3         2m
default-token-xkp6t   kubernetes.io/service-account-token   3         52m
tls-kiteci-com        kubernetes.io/tls                     2         44s
```

This process updates the Ingress with an ACME path. You can see the updated Ingress in `./deploy/ing-with-acme-path.yaml` .

- Now, update the Ingress to add `spec.tls` section to activate TLS for both api and ws endpoints.

```console
$ kubectl apply -f ./deploy/ing-with-tls.yaml
ingress "voyager-demo" configured
```

```yaml
apiVersion: voyager.appscode.com/v1beta1
kind: Ingress
metadata:
  name: voyager-demo
  namespace: default
spec:
  tls:
  - hosts:
    - api.kiteci.com
    - ws.kiteci.com
    ref:
      kind: Certificate
      name: kiteci-com
  rules:
  - host: api.kiteci.com
    http:
      paths:
      - backend:
          serviceName: hello-grpc
          servicePort: '80'
  - host: ws.kiteci.com
    http:
      paths:
      - backend:
          serviceName: hello-websocket
          servicePort: '80'
```

![wss-1](/images/secure-wss-1.png)
![wss-2](/images/secure-wss-2.png)


- Now, setup the website for kiteci.com

```console
kubectl run nginx --image=nginx
kubectl expose deployment nginx --name=web --port=80 --target-port=80
```

