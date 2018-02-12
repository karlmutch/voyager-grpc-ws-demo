- Deploy a grpc server

```console
$ kubectl apply -f https://raw.githubusercontent.com/appscode/hello-grpc/0.1.0/hack/deploy/deploy.yaml

deployment "hello-grpc" created
service "hello-grpc" created
```

- Deploy a websocket server

```console
$ kubectl apply -f https://raw.githubusercontent.com/appscode/hello-websocket/0.1.0/hack/deploy/deploy.yaml

deployment "hello-websocket" created
service "hello-websocket" created
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
curl -fsSL https://raw.githubusercontent.com/appscode/voyager/6.0.0-alpha.0/hack/deploy/voyager.sh \
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

```console
kubectl create secret generic acme-account \
  --from-literal=ACME_EMAIL=tamal@appscode.com \
  --from-literal=ACME_SERVER_URL=https://acme-staging.api.letsencrypt.org/directory
```

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

- Now update the ingress to add a path for kiteci.com and add `kiteci.com` to tls hosts list.

```console
$ curl -vv -k https://kiteci.com/
*   Trying 35.194.36.183...
* Connected to kiteci.com (35.194.36.183) port 443 (#0)
* found 148 certificates in /etc/ssl/certs/ca-certificates.crt
* found 597 certificates in /etc/ssl/certs
* ALPN, offering http/1.1
* SSL connection using TLS1.2 / ECDHE_RSA_AES_128_GCM_SHA256
*    server certificate verification SKIPPED
*    server certificate status verification SKIPPED
*    common name: kiteci.com (matched)
*    server certificate expiration date OK
*    server certificate activation date OK
*    certificate public key: RSA
*    certificate version: #3
*    subject: CN=kiteci.com
*    start date: Mon, 12 Feb 2018 07:07:56 GMT
*    expire date: Sun, 13 May 2018 07:07:56 GMT
*    issuer: CN=Fake LE Intermediate X1
*    compression: NULL
* ALPN, server accepted to use http/1.1
> GET / HTTP/1.1
> Host: kiteci.com
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.13.8
< Date: Mon, 12 Feb 2018 08:12:03 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 26 Dec 2017 11:11:22 GMT
< ETag: "5a422e5a-264"
< Accept-Ranges: bytes
< Strict-Transport-Security: max-age=15768000
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host kiteci.com left intact
```


- Now to redirect www.kiteci.com to kiteci.com, add the frontendRules mentioned in ./deploy/ing-final.yaml

```
spec:
  frontendRules:
  - port: 80
    rules:
    - acl is_proxy_https hdr(X-Forwarded-Proto) https
    - acl has_www hdr(host) -i www.kiteci.com
    - acl has_www hdr(host) -i www.kiteci.com:80
    - http-request redirect code 308 scheme https if ! is_proxy_https has_www
  - port: 443
    rules:
    - acl has_www hdr(host) -i www.kiteci.com
    - acl has_www hdr(host) -i www.kiteci.com:443
    - http-request set-header Host kiteci.com if has_www
    - http-request redirect code 308 scheme https if has_www
```
