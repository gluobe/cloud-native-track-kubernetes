# Lab 02 - Nodes

A node is a worker machine in Kubernetes, previously known as a minion. A node
may be a VM or physical machine, depending on the cluster. Each node contains
the services necessary to run pods and is managed by the master components. The
services on a node include the container runtime, kubelet and kube-proxy.

## Task 1: Listing nodes

To see which nodes are part of your Kubernetes cluster run the
`kubectl get nodes` command:

```
kubectl get nodes

NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   4h56m   v1.13.4
```

As we are using minikube we only have a single node.  Below is the output of a
3 node cluster:

```
kubectl get nodes

NAME                                        STATUS    ROLES     AGE       VERSION
gke-kbc-steven-default-pool-b82ee1c9-5n9j   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-6wnx   Ready     <none>    20m       v1.11.7-gke.4
gke-kbc-steven-default-pool-b82ee1c9-x13z   Ready     <none>    20m       v1.11.7-gke.4
```

We can use the `kubectl get nodes -o wide` command to get some additional
information about the nodes:

```
kubectl get nodes -o wide

NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   21h   v1.13.4   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://18.6.2
```

## Task 2: Getting detailed node information

If we want more detailed information about a node we have to use the
`kubectl describe node <node_name>` command:

```
Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=minikube
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 13 Mar 2019 11:41:06 +0100
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Thu, 14 Mar 2019 09:28:46 +0100   Wed, 13 Mar 2019 11:41:02 +0100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Thu, 14 Mar 2019 09:28:46 +0100   Wed, 13 Mar 2019 11:41:02 +0100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Thu, 14 Mar 2019 09:28:46 +0100   Wed, 13 Mar 2019 11:41:02 +0100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Thu, 14 Mar 2019 09:28:46 +0100   Wed, 13 Mar 2019 11:41:02 +0100   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    minikube
Capacity:
 cpu:                2
 ephemeral-storage:  16888216Ki
 hugepages-2Mi:      0
 memory:             2038624Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  15564179840
 hugepages-2Mi:      0
 memory:             1936224Ki
 pods:               110
System Info:
 Machine ID:                 09cffa828b994b05afa9f2355d79bc40
 System UUID:                E8B09A49-00D6-470C-8F95-73077565B18B
 Boot ID:                    38f4793c-9937-4cf6-b4ff-5973e97ad6b1
 Kernel Version:             4.15.0
 OS Image:                   Buildroot 2018.05
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.6.2
 Kubelet Version:            v1.13.4
 Kube-Proxy Version:         v1.13.4
Non-terminated Pods:         (19 in total)
  Namespace                  Name                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                ------------  ----------  ---------------  -------------  ---
  default                    container-info-6d9747978f-lrt4s     0 (0%)        0 (0%)      0 (0%)           0 (0%)         20h
  default                    container-info-6d9747978f-vbfbp     0 (0%)        0 (0%)      0 (0%)           0 (0%)         20h
  default                    container-info-6d9747978f-zv7hw     0 (0%)        0 (0%)      0 (0%)           0 (0%)         20h
  kube-system                coredns-86c58d9df4-gp858            100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)     21h
  kube-system                coredns-86c58d9df4-vgqf8            100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)     21h
  kube-system                etcd-minikube                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         21h
  kube-system                kube-addon-manager-minikube         5m (0%)       0 (0%)      50Mi (2%)        0 (0%)         21h
  kube-system                kube-apiserver-minikube             250m (12%)    0 (0%)      0 (0%)           0 (0%)         21h
  kube-system                kube-controller-manager-minikube    200m (10%)    0 (0%)      0 (0%)           0 (0%)         19h
  kube-system                kube-proxy-48x92                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         19h
  kube-system                kube-scheduler-minikube             100m (5%)     0 (0%)      0 (0%)           0 (0%)         21h
  kube-system                storage-provisioner                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         21h
  lab-06                     container-info-5776f489fb-lvbjx     0 (0%)        0 (0%)      0 (0%)           0 (0%)         17h
  lab-07                     container-info-6d9747978f-2x5r7     0 (0%)        0 (0%)      0 (0%)           0 (0%)         20h
  lab-07                     container-info-6d9747978f-c9twz     0 (0%)        0 (0%)      0 (0%)           0 (0%)         20h
  lab-07                     container-info-6d9747978f-sqvtw     0 (0%)        0 (0%)      0 (0%)           0 (0%)         20h
  lab-09                     meme-persistent-84cdc97446-bljrg    0 (0%)        0 (0%)      0 (0%)           0 (0%)         17h
  lab-09                     meme-persistent-84cdc97446-rftlm    0 (0%)        0 (0%)      0 (0%)           0 (0%)         17h
  lab-09                     meme-persistent-84cdc97446-wjgjk    0 (0%)        0 (0%)      0 (0%)           0 (0%)         17h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                755m (37%)   0 (0%)
  memory             190Mi (10%)  340Mi (17%)
  ephemeral-storage  0 (0%)       0 (0%)
Events:
  Type    Reason    Age    From                  Message
  ----    ------    ----   ----                  -------
  Normal  Starting  3m19s  kube-proxy, minikube  Starting kube-proxy.
```

Read through the output above to see all the information that is available about
the node.

The `kubectl describe node <node_name>` is a very useful tool when
troubleshooting a node that is failing.  Going through the output will, in
almost all cases, give a  good indication why the node is not behaving as it
should.