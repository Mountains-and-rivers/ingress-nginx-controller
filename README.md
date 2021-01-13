# controller部署验证

```
 kubectl apply -f ingress-nginx-controller.yaml
```

```
kubectl get pod -n ingress-nginx
 NAME                                        READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-6bc9f8bb7d-ld6wx   1/1     Running   0          15m
```

# 部署service nodePort

```
kubectl apply -f service-nodePort.yaml
```

```
kubectl get service -n ingress-nginx

NAME            TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.0.0.220   <none>        80:31856/TCP,443:30619/TCP   17m
```

# 验证

```
curl -i localhost:80/healthz

HTTP/1.1 200 OK
Server: openresty/1.15.8.1
Date: Wed, 13 Jan 2021 13:04:59 GMT
Content-Type: text/html
Content-Length: 0
Connection: keep-alive

```



quay.io/kubernetes-ingress-controller/nginx-ingress-controller :0.25.0 镜像分片 20个
