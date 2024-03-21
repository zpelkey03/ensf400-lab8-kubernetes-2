# Steps to create assignment 3 requirments and outputs

1. To start the enviornment run 
```
$ minikube start
```

2. Enable ingress for kubernetes to use 
```
$ minikube addons enable ingress
$ kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx   ingress-nginx-admission-create-kkcz4        0/1     Completed   0          2m29s
ingress-nginx   ingress-nginx-admission-patch-5g8zb         0/1     Completed   0          2m29s
ingress-nginx   ingress-nginx-controller-7c6974c4d8-d2cmv   1/1     Running     0          2m29s

```
3.  Apply all the files to be run 

```
$ kubectl apply -f .
deployment.apps/app-1-dep created
ingress.networking.k8s.io/app-1-ingress unchanged
service/app-1-svc created
deployment.apps/app-2-dep created
ingress.networking.k8s.io/app-2-ingress unchanged
service/app-2-svc created
configmap/nginx-configmap configured
deployment.apps/nginx-dep created
ingress.networking.k8s.io/nginx-ingress unchanged
service/nginx-svc created
```

4. Run get pods to ceck if pods are running
```
@zpelkey03 ➜ /workspaces/ensf400-lab8-kubernetes-2/assignment3 (main) $ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
app-1-dep-bd7f64bbd-2fnnq    1/1     Running   0          37m
app-1-dep-bd7f64bbd-9c6d5    1/1     Running   0          37m
app-1-dep-bd7f64bbd-fw4hb    1/1     Running   0          37m
app-2-dep-68bdd5555-74nnl    1/1     Running   0          37m
app-2-dep-68bdd5555-9qfrn    1/1     Running   0          37m
app-2-dep-68bdd5555-rvwdm    1/1     Running   0          37m
nginx-dep-84bcd8f686-88b6v   1/1     Running   0          37m
nginx-dep-84bcd8f686-c6z2m   1/1     Running   0          37m
nginx-dep-84bcd8f686-gt8qs   1/1     Running   0          37m
nginx-dep-84bcd8f686-h5ftw   1/1     Running   0          37m
nginx-dep-84bcd8f686-q9hzf   1/1     Running   0          37m
```
4. Get your IP
```
$ minikube ip
192.168.49.2
```

5. Curl nginx, app-1, app-2, and app-3 with that IP address
```
$ curl -kL http://192.168.49.2/nginx
<html>
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>

$ curl -kL http://192.168.49.2/app-1
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

$ curl -kL http://192.168.49.2/app-2
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>

$ curl -kL http://192.168.49.2/app-3
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
