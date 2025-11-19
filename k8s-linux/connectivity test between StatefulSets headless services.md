## This guide will provide step of instructions to evaluate the connectivity test between StatefulSets headless services

#### Deployed two StatefulSets and corresponding headless services n perform the connectivity check:

- StatefulSets-0 deployment:
```
$ kubectl apply -f - <<EOF
> apiVersion: v1
kind: Service
metadata:
  name: headless-svc-0
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: statefulsets-0
spec:
  serviceName: "nginx"
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.21
        ports:
        - containerPort: 80
          name: web
> EOF
service/headless-svc-0 created
statefulset.apps/statefulsets-0 created
```
- StatefulSets-1 deployment:
```
$ kubectl apply -f - <<EOF
> apiVersion: v1
kind: Service
metadata:
  name: headless-svc-1
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: statefulsets-1
spec:
  serviceName: "nginx"
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.21
        ports:
        - containerPort: 80
          name: web
> EOF
service/headless-svc-1 created
statefulset.apps/statefulsets-1 created
```
- StatefulSets, pod, svc statuses:
```
$ kubectl get statefulsets,po,svc
NAME                              READY   AGE
statefulset.apps/statefulsets-0   1/1     3m1s
statefulset.apps/statefulsets-1   1/1     85s

NAME                   READY   STATUS    RESTARTS   AGE
pod/statefulsets-0-0   1/1     Running   0          3m1s
pod/statefulsets-1-0   1/1     Running   0          85s

NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/headless-svc-0   ClusterIP   None         <none>        80/TCP    3m1s
service/headless-svc-1   ClusterIP   None         <none>        80/TCP    85s
```

- Headless service connectivity test:
```
$ kubectl exec -it pod/statefulsets-0-0 -- bash
root@statefulsets-0-0:/# curl headless-svc-1
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
```
