# Lab 10 - Lists

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-10

---

namespace "lab-10" created
```

## Task 1: Creating objects using a list

You can put different Kubernetes objects in a list, and apply the list instead 
of applying the objects seperatly. This will deploy all the resources you have 
listed in this list.

Create the file `lab-10-list.yml` with the following content, the items in the 
list should look familiar as we have created them before individually.

```
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: Service
  metadata:
    name: container-info
  spec:
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
    type: NodePort
    selector:
      app: container-info
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: container-info
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
      spec:
        containers:
          - name: container-info
            image: gluobe/container-info:green
```

We can apply the entire list in one command.

```
kubectl apply -f lab-10-list.yml -n lab-10

---

service/container-info created
deployment.apps/container-info created
```

Now we can list all the resources we have created.

```
kubectl get all -n lab-10

---

NAME                                  READY   STATUS    RESTARTS   AGE
pod/container-info-56f64f4f8c-fxrtb   1/1     Running   0          30s
pod/container-info-56f64f4f8c-m29hv   1/1     Running   0          30s
pod/container-info-56f64f4f8c-s5r4c   1/1     Running   0          30s

NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/container-info   NodePort   10.102.28.105   <none>        80:31432/TCP   30s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/container-info   3/3     3            3           30s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/container-info-56f64f4f8c   3         3         3       30s
```

And like before we can access this deployment with the following command.

```
minikube service container-info -n lab-10
```

## Task 2: Cleanup

Clean up the namespace for this lab:

```
kubectl delete ns lab-10

---

namespace "lab-10" deleted
```
