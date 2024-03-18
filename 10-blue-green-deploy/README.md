# 10 - Blue-Green Deployment

## What is Blue-Green Deployment ? 
Blue-Green deployment is an application deployment strategy using which we can easily update one version named as BLUE while the another version named as GREEN will keep serving the request once done we can switch back to BLUE if required . It has great advantage for real production scenarios, no downtime is required and guess what we can easily switch to older version whenever required .

# How Blue-Green Deployment works ? 
This can be easily archived using labels and selectors; we will mostly use kubectl patch command as below, Note: we can even do this manually by editing service.

```bash
$ kubectl patch service  SERVICENAME -p '{"spec":{"selector":{"KEY": "VALUE"}}}'
``` 
In this example we will create two pods with the httpd image and will change the “It Works “message to “It Works – Blue Deployment” and “It Works – Green Deployment” for second Pod. We will also create a service which will map to blue first and once the update is done we will patch it to green. 

# Creating a Pod with Labels

```bash
cd /workspaces/ensf400-lab8-kubernetes-2/10-blue-green-deploy
```

```bash
$ kubectl apply -f blue.yml
pod/bluepod created
```

```bash
$ kubectl get pods --show-labels
NAME           READY   STATUS    RESTARTS   AGE   LABELS
bluepod        1/1     Running   0          9s    app=blue

$ kubectl apply -f green.yml 
pod/greenpod created
```

```bash
$ kubectl get pods --show-labels
NAME           READY   STATUS    RESTARTS   AGE   LABELS
bluepod        1/1     Running   0          43s   app=blue
greenpod       1/1     Running   0          11s   app=green
```

Create a file `svc.yml` with the following config:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: testing
  name: myapp
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: blue
```
In the above service YAML file, we are mapping the service to our `blue` pod via selectors. 
Apply the service above:
```bash
$ kubectl apply -f svc.yml
service/myapp created
```

Just for understanding purpose we are changind, the default landing page of our httpd application

```bash
$ kubectl exec -it bluepod -- bash
root@bluepod:/usr/local/apache2# echo "Hello from Blue-Pod" >> htdocs/index.html
exit
```

```bash
$ kubectl exec -it greenpod -- bash
root@greenpod:/usr/local/apache2# echo "Hello from Green-Pod" >> htdocs/index.html
exit
```

We can verify if both the pods are having the updated output or not. 

```bash
# get the internal IP addresses of the blue and green pods
$ kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
bluepod        1/1     Running   0          3m28s   10.244.2.2   ensf400-m03   <none>           <none>
greenpod       1/1     Running   0          2m56s   10.244.2.3   ensf400-m03   <none>           <none>
```

# Access the HTTP service by using blue pod's IP.
Run a `curl` pod to test the access to the blue and green pod using their IP addresses above:
```bash
kubectl run -it --rm --image=curlimages/curl curly -- sh

$ curl 10.244.2.2
<html><body><h1>It works!</h1></body></html>
Hello from Blue-Pod

# Access the HTTP service by using green pod's IP.
$ curl 10.244.2.3
<html><body><h1>It works!</h1></body></html>
Hello from Green-Pod

# exit the curl pod
$ exit 
Session ended, resume using 'kubectl attach curly -c curly -i -t' command when the pod is running
pod "curly" deleted
```

Now let's see how it works with service IP 
```bash
$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   4d
myapp        ClusterIP   10.106.8.206   <none>        80/TCP    5m59s
```

Run a `curl` pod again to test the access to the service:
```bash
$ kubectl run -it --rm --image=curlimages/curl curly -- sh

$ curl myapp
<html><body><h1>It works!</h1></body></html>
Hello from Blue-Pod

# exit the curl pod
$ exit
Session ended, resume using 'kubectl attach curly -c curly -i -t' command when the pod is running
pod "curly" deleted
```

Let's try to switch to our green deployment by changing the service mapping using below command, if we try to curl the service IP it should take us to the green pod. 

```bash
$ kubectl patch service myapp -p '{"spec":{"selector":{"app": "green"}}}'
service/myapp patched
```

Run a `curl` pod again to test the access to the service:
```bash
$ kubectl run -it --rm --image=curlimages/curl curly -- sh

$ curl myapp
<html><body><h1>It works!</h1></body></html>
Hello from Green-Pod

# exit the curl pod
$ exit
Session ended, resume using 'kubectl attach curly -c curly -i -t' command when the pod is running
pod "curly" deleted
```

In this way, we can conclude that simply patching a service selector will enable the switch of the backend pods to seamlessly update the application without downtime.

