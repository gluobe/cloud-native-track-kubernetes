# Lab 03 - Namespaces

Kubernetes supports multiple virtual clusters backed by the same physical
cluster. These virtual clusters are called namespaces.

Namespaces make it possible to run different environments on a single Kubernetes
cluster, such as DEV, TST and UAT.  Namespace can also be used to set different
access controls per namespace.

Most objects in Kubernetes can be namespaced (pods, services, pvc,...), keep in
mind however that some objects however cannot be namespaced and are cluster-wide
(pv for example).

## Task 1: Listing namespaces

To see which namespaces are available use the `kubectl get namespaces` command:

```
kubectl get namespaces

---

NAME          STATUS    AGE
default       Active    40m
kube-public   Active    40m
kube-system   Active    40m
```

## Task 2: Creating a new namespace

Creating a namespace is easy, `kubectl create namespace <namespacename>`:

```
kubectl create namespace test

---

namespace "test" created
```

Check that your namespace has been created:

```
kubectl get ns

---

NAME           STATUS    AGE
default        Active    45m
kube-public    Active    45m
kube-system    Active    45m
test           Active    1m
```

> NOTE: as you can see we abreviated `namespace` to `ns`, most of the objects in
> Kubernetes have abreviations that you can use on the command line

## Task 3: Specifying namespaces

When working with namespaces it is important to know that if you do not specify
a specific namepace when issuing a `kubectl` command the default namespace of
your current context is assumed.  In our case this will be `default` but you can
of course create new and/or customize your context.

This means that running the following command:

```
kubectl get all

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

Is exactly the same as running:

```
kubectl get all -n default

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

But if we do this for a different namespace we of course get a completely
different result:

```
kubectl get all -n kube-system

---

NAME                                   READY   STATUS    RESTARTS   AGE
pod/coredns-86c58d9df4-gp858           1/1     Running   2          21h
pod/coredns-86c58d9df4-vgqf8           1/1     Running   3          21h
pod/etcd-minikube                      1/1     Running   2          21h
pod/kube-addon-manager-minikube        1/1     Running   3          21h
pod/kube-apiserver-minikube            1/1     Running   2          21h
pod/kube-controller-manager-minikube   1/1     Running   1          19h
pod/kube-proxy-48x92                   1/1     Running   2          19h
pod/kube-scheduler-minikube            1/1     Running   2          21h
pod/storage-provisioner                1/1     Running   3          21h

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   21h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/kube-proxy   1         1         1       1            1           <none>          21h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           21h

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-86c58d9df4   2         2         2       21h
```

## Task 4: All namespaces

It could happen that you do not really know in which namespace a Kubernetes
object is running.  If that is the case you can always add the
`--all-namespaces` option to your command to list objects from all of the
namespaces.

```
kubectl get pods --all-namespaces

NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-86c58d9df4-gp858           1/1     Running   2          21h
kube-system   coredns-86c58d9df4-vgqf8           1/1     Running   3          21h
kube-system   etcd-minikube                      1/1     Running   2          21h
kube-system   kube-addon-manager-minikube        1/1     Running   3          21h
kube-system   kube-apiserver-minikube            1/1     Running   2          21h
kube-system   kube-controller-manager-minikube   1/1     Running   1          19h
kube-system   kube-proxy-48x92                   1/1     Running   2          19h
kube-system   kube-scheduler-minikube            1/1     Running   2          21h
kube-system   storage-provisioner                1/1     Running   3          21h
```

## Task 5: Changing the default context

As mentioned above, we can can change the behaviour that when no namespace is
specified, the default namespace is assumed.  This is done by changing the
context.

Standard behaviour is that default namespace is automatically assumed, let us
verify this first:

```
kubectl get all

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

Now change the "default" namespace to `kube-system`:

```
kubectl config set-context --current --namespace=kube-system

---

Context "minikube" modified.
```

If we now run `kubectl get all` without specifying a namespace we get all the
objects of the `kube-system` namespace:

```
kubectl get all

---

NAME                                        READY   STATUS    RESTARTS   AGE
pod/coredns-fb8b8dccf-269qq                 1/1     Running   3          49d
pod/coredns-fb8b8dccf-sd9qp                 1/1     Running   2          49d
pod/etcd-minikube                           1/1     Running   1          49d
pod/kube-addon-manager-minikube             1/1     Running   3          49d
pod/kube-apiserver-minikube                 1/1     Running   2          49d
pod/kube-controller-manager-minikube        1/1     Running   0          6m26s
pod/kube-proxy-z92f5                        1/1     Running   1          49d
pod/kube-scheduler-minikube                 1/1     Running   2          49d
pod/kubernetes-dashboard-79dd6bfc48-rfwqb   1/1     Running   1          49d
pod/storage-provisioner                     1/1     Running   2          49d


NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   49d
service/kubernetes-dashboard   ClusterIP   10.105.148.181   <none>        80/TCP                   49d

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/kube-proxy   1         1         1       1            1           <none>          49d

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns                2/2     2            2           49d
deployment.apps/kubernetes-dashboard   1/1     1            1           49d

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-fb8b8dccf                 2         2         2       49d
replicaset.apps/kubernetes-dashboard-79dd6bfc48   1         1         1       49d
```

Of course we can still get the objects from the default namespace by specifying
it specifically:

```
kubectl get all -n default

---

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

Now change the context again to its normal behavior:

```
kubectl config set-context --current --namespace=default

---

Context "minikube" modified.
```

## Task 6: Deleting namespaces

Deleting a namespace is very easy, keep in mind however that when you delete a
namespace *all* the objects in that namespace will be deleten. So always verify
that all the objects in that namespace can be deleted:

```
kubectl delete ns test

---

namespace "test" deleted
```
