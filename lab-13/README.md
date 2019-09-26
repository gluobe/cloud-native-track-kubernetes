# Lab 13 - Node labeling

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-13

---

namespace "lab-13" created
```

## Task 1: Labeling a node

The basic node labeling will give you the option to restrict pods to specific 
nodes. This could be very usefull if you are running multiple environments. 
Imagine that we are pushing our application from `test` to `uat` to `prod`. With 
node labeling you can specify for example that all the pods of the `test` 
environment are going to be scheduled on the `test` node.

Because we are using `minikube` in this lab, we only have 1 node. This example 
will show you how you can label the `minikube` node and make sure that the 
`application` is running on this `minikube` node.

First we need to find the name of the node.

```
kubectl get nodes

---

NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    2d        v1.13.3
```

Now you can add a label to the `minikube` node with the following command.

```
kubectl label nodes minikube environment=test

---

node/minikube labeled
```

This labels the `minikube` node with the `environment=test` label. You can 
confirm that the node is labeled with the command.

```
kubectl get nodes --show-labels

---

NAME       STATUS    ROLES     AGE       VERSION   LABELS
minikube   Ready     master    2d        v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,environment=test,kubernetes.io/hostname=minikube,node-role.kubernetes.io/master=
```

Create a new deployment file `lab-13-label-deployment.yml` and add this content 
to it.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-blue
  labels:
    app: container-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
        version: "blue"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
      nodeSelector:
        environment: test
```

This is basically our standard container-info deployment. We only added the 
following:

```
nodeSelector:
  environment: test
```

And this will define that the deployment is going to run on this specific node.

```
kubectl apply -f lab-13-label-deployment.yml -n lab-13

---

deployment.apps/container-info-blue created
```

You can verify on which node a pod is running using the following command:

```
kubectl get pods -n lab-13 -o wide

---

NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
container-info-blue-689dd4c865-4dhh6   1/1     Running   0          47s   172.17.0.5   minikube   <none>           <none>
container-info-blue-689dd4c865-9bnc8   1/1     Running   0          47s   172.17.0.4   minikube   <none>           <none>
container-info-blue-689dd4c865-r5djk   1/1     Running   0          47s   172.17.0.7   minikube   <none>           <none>
```

As we only have a single node it is obvious that our pods are running on our 
node, but if we would have had more nodes you could see that our pod was running 
on the node we specifically selected.

### Task 2: Pod Affinity

Pod Affinity means that you can schedule pods with the same label on the same
node. If available. So imagine that we want our `test` pod to run next to other
pods with the label `test` on a node. We need to define the `podAffinity` like
this.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - test
        topologyKey: failure-domain.beta.kubernetes.io/zone
  containers:
  - name: container-info
    image: gluobe/container-info:blue
```

We are using `requiredDuringSchedulingIgnoredDuringExecution` this means that
this is a requirement while the pod is getting scheduled. It's also possible to
define `preferredDuringSchedulingIgnoredDuringExecution`. This means that he
will try to schedule the pod with these conditions if possible. Otherwise it
will schedule the pod on a other node.

### Task 3: Pod Anti Affinity

The oposite from what we did above can be done with `antiAffinity`. If we do the
following in the pod yml.

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - test
        topologyKey: failure-domain.beta.kubernetes.io/zone
  containers:
  - name: container-info
    image: gluobe/container-info:blue
```

The pod will `not` schedule next to the already running `test` environment pod
on any node. This means that the scheduler is going to look for another pod
where no pod is running with the label `environment=test`.

## Task 4: Cleanup

Clean up the namespace for this lab:

```
kubectl delete ns lab-13

---

namespace "lab-13" deleted
```