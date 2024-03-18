# 8 - Configmap

This scenario shows:
- how to create config map (declerative way),
- how to use configmap: volume and environment variable,
- how to create configmap with command (imperative way),
- how to get/delete configmap


## Steps

Create a YAML file (`config.yaml`) in your directory and copy the below definition into the file.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap               
data:
  db_server: "db.example.com"        # configmap key-value parameters
  database: "mydatabase"
  site.settings: |
    color=blue
    padding:25px
```
Then apply this config:
```bash
$ kubectl apply -f configmap.yaml
configmap/myconfigmap created
```
Verify that the configmap is created by

```bash
$ kubectl get configmap
```

Create another YAML file (`pod.yaml`) in your directory and copy the below definition into the file.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmappod
spec:
  containers:
  - name: configmapcontainer
    image: nginx
    env:                             # configmap using environment variable
      - name: DB_SERVER
        valueFrom:
          configMapKeyRef:           
            name: myconfigmap        # configmap name, from "myconfigmap" 
            key: db_server
      - name: DATABASE
        valueFrom:
          configMapKeyRef:
            name: myconfigmap
            key: database
    volumeMounts:
      - name: config-vol
        mountPath: "/config"
        readOnly: true
  volumes:
    - name: config-vol               # transfer configmap parameters using volume
      configMap:
        name: myconfigmap
```

Next, apply the Pod config referring to the configmap:
```bash
$ kubectl apply -f pod.yaml
pod/configmappod created
```

If we run the terminal of the pod and print its environemental variables, we will see the env values are populated from the configmap:

```bash
$ kubectl exec -it configmappod -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=configmappod
TERM=xterm
DB_SERVER=db.example.com
DATABASE=mydatabase
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
NGINX_VERSION=1.25.4
NJS_VERSION=0.8.3
PKG_RELEASE=1~bookworm
HOME=/root
```

We can also define a configmap with imperative way (--from-file and --from-literal) (create a file `theme.txt` and write a line `theme=dark`)

```bash
$ kubectl create configmap myconfigmap2 --from-literal=background=red --from-file=theme.txt
configmap/myconfigmap2 created
```
Then, we can view the content of the configmap and verify if these values are present:
```yaml
$ kubectl get configmap myconfigmap2 -o yaml
apiVersion: v1
data:
  background: red
  theme.txt: theme=dark
kind: ConfigMap
metadata:
  creationTimestamp: "2024-03-18T06:40:01Z"
  name: myconfigmap2
  namespace: default
  resourceVersion: "12039"
  uid: f39ddff3-99c1-4589-ab65-e9a10d0ef706
```
We can see that both the literal value and the file content has been loaded into `myconfigmap2`.

At last, delete configmap:

```bash
$ kubectl delete configmap myconfigmap2
```
