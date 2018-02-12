- Deploy a grpc server

```console
kubectl apply -f https://raw.githubusercontent.com/appscode/hello-grpc/0.1.0/hack/deploy/deploy.yaml
```

- Deploy a websocket server

```console
kubectl apply -f https://raw.githubusercontent.com/appscode/hello-websocket/0.1.0/hack/deploy/deploy.yaml
```


- Install Voyager Operator
```console
curl -fsSL https://raw.githubusercontent.com/appscode/voyager/5.0.0-rc.11/hack/deploy/voyager.sh \
    | bash -s -- --provider=gke
```