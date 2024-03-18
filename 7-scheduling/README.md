# 7 - Scheduling

## Using Multi-Node Clusters in Minikube

We will first start a multi-node cluster on minikube. Start a cluster with 3 nodes in the driver of your choice:

```shell
$ minikube start --nodes 3 -p ensf400
😄  [ensf400] minikube v1.32.0 on Ubuntu 20.04 (docker/amd64)
✨  Automatically selected the docker driver. Other choices: none, ssh
📌  Using Docker driver with root privileges
👍  Starting control plane node ensf400 in cluster ensf400
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.28.3 preload ...
    > preloaded-images-k8s-v18-v1...:  403.35 MiB / 403.35 MiB  100.00% 87.29 M
    > gcr.io/k8s-minikube/kicbase...:  453.90 MiB / 453.90 MiB  100.00% 61.58 M
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner

👍  Starting worker node ensf400-m02 in cluster ensf400
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
    ▪ env NO_PROXY=192.168.49.2
🔎  Verifying Kubernetes components...

👍  Starting worker node ensf400-m03 in cluster ensf400
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2,192.168.49.3
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
    ▪ env NO_PROXY=192.168.49.2
    ▪ env NO_PROXY=192.168.49.2,192.168.49.3
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "ensf400" cluster and "default" namespace by default
```

Get the list of your nodes:

```shell
$ kubectl get nodes
```
```
NAME          STATUS   ROLES           AGE   VERSION
ensf400       Ready    control-plane   98s   v1.28.3
ensf400-m02   Ready    <none>          63s   v1.28.3
ensf400-m03   Ready    <none>          34s   v1.28.3
```

- You can also check the status of your nodes:

```shell
$ minikube status -p ensf400
```

```
ensf400
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

ensf400-m02
type: Worker
host: Running
kubelet: Running

ensf400-m03
type: Worker
host: Running
kubelet: Running
```

## What is Kubernetes Scheduling?

The Kubernetes Scheduler is a core component of Kubernetes: After a user or a controller creates a Pod, the Kubernetes Scheduler, monitoring the Object Store for unassigned Pods, will assign the Pod to a Node. Then, the Kubelet, monitoring the Object Store for assigned Pods, will execute the Pod.

## What is a Scheduler for?

<p align="center">
  <img src='schedulerhow.png'>
</p>

The Kubernetes scheduler is in charge of scheduling pods onto nodes. Basically it works like this:

   1. You create a pod
   1. The scheduler notices that the new pod you created doesn’t have a node assigned to it
   1. The scheduler assigns a node to the pod

It’s not responsible for actually running the pod – that’s the kubelet’s job. So it basically just needs to make sure every pod has a node assigned to it. Easy, right?

Kubernetes in general has this idea of a "controller". A controller’s job is to:

  - Look at the state of the system
  - Notice ways in which the actual state does not match the desired state (like “this pod needs to be assigned a node”)
  - Repeat

The scheduler is a kind of controller. There are lots of different controllers and they all have different jobs and operate independently.

## What is node affinity ?

In simple words this allows you to tell Kubernetes to schedule pods only to specific subsets of nodes.

The initial node affinity mechanism in early versions of Kubernetes was the nodeSelector field in the pod specification. The node had to include all the labels specified in that field to be eligible to become the target for the pod.

## nodeSelector
Apply the following configuration to Kubernetes to create the pod `nginx`.

```bash
$ cd /workspaces/ensf400-lab8-kubernetes-2/7-scheduling

$ kubectl label nodes ensf400-m02 mynode=worker-1
node/ensf400-m02 labeled

$ kubectl apply -f pod-nginx.yaml
pod/nginx created
```

Note that there is a `nodeSelector` configured that will decide on the placement of node for this pod.
```yaml
nodeSelector:
  mynode: worker-1
```

We have label on the node with node name, in this case Kubernetes scheduler will schedule the pod to Node `ensf400-m02` as it has the `mynode=worker-1`. 

```bash
kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          55s   10.244.1.2   ensf400-m02   <none>           <none>
```
We can see that the NODE for Pod `nginx` is `ensf400-m02`. This can be further verified by describing the pod.
```bash
$ kubectl describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             ensf400-m02/192.168.49.3
Start Time:       Thu, 14 Mar 2024 07:14:19 +0000
Labels:           env=test
Annotations:      <none>
Status:           Running
IP:               10.244.1.2
IPs:
  IP:  10.244.1.2
Containers:
  nginx:
    Container ID:   docker://d9beff7fa2cc97d103587dc084578543c39f97b883eb27ed77480b3e3548d249
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:6db391d1c0cfb30588ba0bf72ea999404f2764febf0f1f196acd5867ac7efa7e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 14 Mar 2024 07:14:29 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4clq8 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-4clq8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              mynode=worker-1
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Normal   Scheduled         2m1s   default-scheduler  Successfully assigned default/nginx to ensf400-m02
  Normal   Pulling           2m     kubelet            Pulling image "nginx"
  Normal   Pulled            111s   kubelet            Successfully pulled image "nginx" in 8.61s (8.61s including waiting)
  Normal   Created           111s   kubelet            Created container nginx
  Normal   Started           111s   kubelet            Started container nginx

```
You can check the above output for `Node-Selectors: mynode=worker-1`.

Deleting the Pod:
```
kubectl delete -f pod-nginx.yaml
pod "nginx" deleted
```
Now, lets test what happens when we have a nodeSelector configured in a way that none of the node meets the requirements. This can often happen when there are no valid node to shedule the pod on. For instance, when the pod is request too many CPUs but there are simply no nodes with sufficent amount of vCPUs available. 

Open `pod-nginx.yaml` and change the `nodeSelector`:
```yaml
nodeSelector: 
  mynode: worker-x
```
Then apply the configuration again:
```bash
$ kubectl apply -f pod-nginx.yaml
pod/nginx created
```
Check the pod status:
```bash
$ kubectl get po -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     Pending   0          5s    <none>   <none>   <none>           <none>
```
We can see that the pod status is `Pending` and the node assigned is `<NONE>`. This is because there is no node with the label `mynode=worker-x`. Therefore, Kubernetes cannot find any node to schedule this pod on. This can be further verified by the pod events:
```bash
$ kubectl describe pod nginx
Name:             nginx
...
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  108s  default-scheduler  0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling..
```
We can see that in the events, there is a warning message telling exactly why `FailedScheduling` happened: because 0 of the 3 nodes matches the Pod's selector.

If we want to get the Pod scheduled, we don't have to create it again. Instead, we can simply add the desired label to a node to meet its node selection criteria:

```bash
$ kubectl label node ensf400-m03 mynode=worker-x
node/ensf400-m03 labeled
```
If we check the pod status again, we can find an update:
```bash
$ kubectl get po -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          8m43s   10.244.2.2   ensf400-m03   <none>           <none>
```
Now the Pod is in `Running` state and the selected Node is `ensf400-m03`, which is the node we just labeled.

We can use nodeSelector to choose one or a group of specific Nodes to schedule. This is particularly useful if the Nodes have unique features such as powerful GPUs or other specialized hardware.

## Node affinity

Node affinity is conceptually similar to nodeSelector – it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node.

There are currently two types of node affinity.
1. `requiredDuringSchedulingIgnoredDuringExecution`: Required during scheduling, ignored during execution; also known as "hard" requirements.
1. `preferredDuringSchedulingIgnoredDuringExecution`: Preferred during scheduling, ignored during execution; also known as "soft" requirements.

Run the following commands to apply the configuration:

```bash
$ kubectl label nodes ensf400-m02 mynode=worker-1
node/ensf400-m02 not labeled

$ kubectl label nodes ensf400-m03 mynode=worker-x
node/ensf400-m03 not labeled

$ kubectl apply -f pod-with-node-affinity.yaml
pod/with-node-affinity created
```



## Viewing Your Pods

```bash
kubectl get pods --output=wide
NAME                 READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
with-node-affinity   1/1     Running   0          72s   10.244.1.3   ensf400-m02   <none>           <none>
```

```
$ kubectl describe po
Name:             with-node-affinity
Namespace:        default
Priority:         0
Service Account:  default
Node:             ensf400-m02/192.168.49.3
Start Time:       Thu, 14 Mar 2024 07:46:50 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.1.3
IPs:
  IP:  10.244.1.3
Containers:
  nginx:
    Container ID:   docker://04f1454d50454f1b817ec4d09a4d386ada381fc9c47d747f44bf8674de12c9aa
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:6db391d1c0cfb30588ba0bf72ea999404f2764febf0f1f196acd5867ac7efa7e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 14 Mar 2024 07:46:53 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nx6l9 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-nx6l9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  100s  default-scheduler  Successfully assigned default/with-node-affinity to ensf400-m02
  Normal  Pulling    99s   kubelet            Pulling image "nginx"
  Normal  Pulled     98s   kubelet            Successfully pulled image "nginx" in 1.273s (1.274s including waiting)
  Normal  Created    97s   kubelet            Created container nginx
  Normal  Started    97s   kubelet            Started container nginx
```


Finally we can clean up the resources created:
```
kubectl delete -f pod-with-node-affinity.yaml
```

# Node Anti-affinity

While Kubernetes scheduler will ensure all of pods get places on healthy nodes with sufficient resources, the scheduler will not take in to account which pods get placed on which nodes based on the application running in the pod. This means you could end up in a situation where all your pods from a particular deployment are all placed on the same node. The downside of such cases is that it puts that application in risk of failure in case that single node goes down.

What we really want is for the pods to be spread out among all the nodes as to increase the size of the point of failure. It’s less likely for multiple nodes to go down all at once than for one node to go down. — We are going to use pod anti-affinity to direct the scheduler to place our pods with a strategy for high availability.

Pod anti-affinity allows us to accomplish the opposite of pod affinity, ensuring certain pods don’t run on the same node as other pods. We are going to use this to make sure our pods that run the same application are spread among multiple nodes. To do this, we will tell the scheduler to not place a pod with a particular label onto a node that contains a pod with the same label.

## Steps
```bash
kubectl apply -f deploy-with-node-anti-affinity.yaml
```
## Viewing Your Pods

```bash
$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx-c4f46d545-2cv59   1/1     Running   0          82s   10.244.0.3   ensf400       <none>           <none>
nginx-c4f46d545-dp88k   1/1     Running   0          82s   10.244.1.2   ensf400-m02   <none>           <none>
nginx-c4f46d545-x72ff   1/1     Running   0          83s   10.244.2.2   ensf400-m03   <none>           <none>
```

As you can see, we have 3 pods with a label `app: nginx`. In our affinity instructions, we specify a `podAntiAffinity` that tells the scheduler to not place this pod on a node with an existing pod with the label `app: nginx`. This ensures that if we will have the pods spread over multiple nodes.


## Step  Cleanup

Finally you can clean up the resources you created in your cluster:
```
kubectl delete -f deploy-with-node-anti-affinity.yaml
```
