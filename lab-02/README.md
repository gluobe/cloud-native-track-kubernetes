# Lab 02 - Nodes

A node is a worker machine in Kubernetes, previously known as a minion. A node
may be a VM or physical machine, depending on the cluster.  A node is where your 
application containers will run.

Each node contains the services necessary to run pods and is managed by the 
master components. The services on a node include the container runtime, kubelet 
and kube-proxy.

## Task 1: Listing nodes

To see which nodes are part of your Kubernetes cluster run the
`kubectl get nodes` command:

```
kubectl get nodes

---

NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   77d   v1.16.0
```

As we are using minikube we only have a single node.  Below is the output of a
3 node cluster:

```
kubectl get nodes

---

NAME                                        STATUS    ROLES     AGE       VERSION
gke-kbc-steven-default-pool-b82ee1c9-5n9j   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-6wnx   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-x13z   Ready     <none>    20m       v1.11.7-gke.4
```

We can use the `kubectl get nodes -o wide` command to get some additional
information about the nodes:

```
kubectl get nodes -o wide

---

NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   77d   v1.16.0   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://18.9.6
```

## Task 2: Getting detailed node information

If we want more detailed information about a node we have to use the
`kubectl describe nodes <node_name>` command, for example:

```
kubectl describe nodes minikube

---

Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=minikube
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 10 Jul 2019 13:29:48 +0200
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Thu, 26 Sep 2019 11:50:39 +0200   Wed, 10 Jul 2019 13:29:43 +0200   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Thu, 26 Sep 2019 11:50:39 +0200   Wed, 10 Jul 2019 13:29:43 +0200   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Thu, 26 Sep 2019 11:50:39 +0200   Wed, 10 Jul 2019 13:29:43 +0200   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Thu, 26 Sep 2019 11:50:39 +0200   Wed, 10 Jul 2019 13:29:43 +0200   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    minikube
Capacity:
 cpu:                2
 ephemeral-storage:  17784772Ki
 hugepages-2Mi:      0
 memory:             2038624Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  16390445849
 hugepages-2Mi:      0
 memory:             1936224Ki
 pods:               110
System Info:
 Machine ID:                 84a95625dace4d7b8313ea0f131bebd1
 System UUID:                2449230B-73F2-456A-B1E8-3A5F61B9B546
 Boot ID:                    93caae44-301d-4799-8ba7-d5fcf30b2e52
 Kernel Version:             4.15.0
 OS Image:                   Buildroot 2018.05
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.9.6
 Kubelet Version:            v1.16.0
 Kube-Proxy Version:         v1.16.0
Non-terminated Pods:         (19 in total)
  Namespace                  Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                          ------------  ----------  ---------------  -------------  ---
  default                    bigcommerce-extension-9f56dd765-6pvgj         0 (0%)        0 (0%)      0 (0%)           0 (0%)         77d
  default                    container-info-c5cf66fb6-9r4gp                0 (0%)        0 (0%)      0 (0%)           0 (0%)         28d
  default                    container-info-c5cf66fb6-q69fs                0 (0%)        0 (0%)      0 (0%)           0 (0%)         28d
  default                    container-info-c5cf66fb6-zrf6w                0 (0%)        0 (0%)      0 (0%)           0 (0%)         28d
  default                    nginx                                         0 (0%)        0 (0%)      0 (0%)           0 (0%)         34d
  default                    nginx2                                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         34d
  default                    nginx3                                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         34d
  kube-system                coredns-5644d7b6d9-cmvbm                      100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)     5m46s
  kube-system                coredns-5644d7b6d9-vbl8f                      100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)     5m47s
  kube-system                etcd-minikube                                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         6m11s
  kube-system                kube-addon-manager-minikube                   5m (0%)       0 (0%)      50Mi (2%)        0 (0%)         6m12s
  kube-system                kube-apiserver-minikube                       250m (12%)    0 (0%)      0 (0%)           0 (0%)         6m12s
  kube-system                kube-controller-manager-minikube              200m (10%)    0 (0%)      0 (0%)           0 (0%)         6m10s
  kube-system                kube-proxy-mkxhk                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         5m40s
  kube-system                kube-scheduler-minikube                       100m (5%)     0 (0%)      0 (0%)           0 (0%)         6m11s
  kube-system                storage-provisioner                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         77d
  kubernetes-dashboard       dashboard-metrics-scraper-76585494d8-xpxpb    0 (0%)        0 (0%)      0 (0%)           0 (0%)         5m47s
  kubernetes-dashboard       kubernetes-dashboard-57f4cb4545-n7ct4         0 (0%)        0 (0%)      0 (0%)           0 (0%)         5m47s
  lab-04                     nginx                                         0 (0%)        0 (0%)      0 (0%)           0 (0%)         28d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                755m (37%)   0 (0%)
  memory             190Mi (10%)  340Mi (17%)
  ephemeral-storage  0 (0%)       0 (0%)
Events:
  Type    Reason                   Age                    From                  Message
  ----    ------                   ----                   ----                  -------
  Normal  Starting                 6m23s                  kubelet, minikube     Starting kubelet.
  Normal  NodeHasSufficientMemory  6m22s (x8 over 6m23s)  kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    6m22s (x8 over 6m23s)  kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     6m22s (x7 over 6m23s)  kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  6m22s                  kubelet, minikube     Updated Node Allocatable limit across pods
  Normal  Starting                 6m                     kube-proxy, minikube  Starting kube-proxy.
  Normal  Starting                 5m37s                  kube-proxy, minikube  Starting kube-proxy.
```

Read through the output above to see all the information that is available about
the node.

The `kubectl describe nodes <node_name>` is a very useful tool when
troubleshooting a node that is failing.  Going through the output (especially 
the event section) will, in almost all cases, give a  good indication why the 
node is not behaving as it should.