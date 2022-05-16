# Lab 07 - Services

A Kubernetes Service is an abstraction which defines a logical set of Pods and a
policy by which to access them - sometimes called a micro-service. The set of
Pods targeted by a Service is (usually) determined by a Label Selector.

As an example, consider an image-processing backend which is running with 3
replicas. Those replicas are fungible - frontends do not care which backend they
use. While the actual Pods that compose the backend set may change, the frontend
clients should not need to be aware of that or keep track of the list of
backends themselves. The Service abstraction enables this decoupling.

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-07

---

namespace "lab-07" created
```

## Task 1: Creating your first service

Create a file `lab-07-deployment.yml` with the YAML for the deployment using the
content below:

```
apiVersion: apps/v1
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
        image: gluobe/container-info:blue
        ports:
        - containerPort: 80
```

Create the deployment using the above file:

```
kubectl apply -f lab-07-deployment.yml -n lab-07

---

deployment "container-info" created
```

Verfiy that this is working:

```
kubectl get pods -n lab-07

---

NAME                              READY     STATUS    RESTARTS   AGE
container-info-5998b79944-gh57z   1/1       Running   0          21s
container-info-5998b79944-t4h6n   1/1       Running   0          21s
container-info-5998b79944-vkmcg   1/1       Running   0          21s
```

Now create a file `lab-07-service.yml` with the YAML for the service using the
content below:

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Create the service using the above file:

```
kubectl apply -f lab-07-service.yml -n lab-07

---

service "container-info" created
```

Check that the service has been created succesfully:

```
kubectl get service -n lab-07

---

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
container-info   NodePort    10.27.247.106   <none>        80/TCP    7m
```

## Task 2: Inspecting your first service

To see more information about the service we can, again, use the
`kubectl describe` command:

```
kubectl describe service container-info -n lab-07

---

Name:			container-info
Namespace:		lab-07
Labels:			<none>
Annotations:		kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"container-info","namespace":"lab-07"},"spec":{"ports":[{"port":80,"protocol":"...
Selector:		app=container-info
Type:			NodePort
IP:			10.103.103.180
Port:			<unset>	80/TCP
NodePort:		<unset>	32093/TCP
Endpoints:		172.17.0.7:80,172.17.0.8:80,172.17.0.9:80
Session Affinity:	None
Events:			<none>
```

From the output you can see that a service looks a lot like a loadbalancer, you
can see the VIP (IP) and the different backends (Endpoints).

## Task 3: Testing your first service

In previous labs we always used the `port-forward` method to expose a pod, as 
this method only allows us to connect to a single pod we will use a different 
method when wofking with service.

To test your service you can run the command below, it will automatically open 
a browser with your service/application:

```
minikube service container-info -n lab-07
```

> NOTE: when you refresh you should see a different "avatar" for each pod, the 
> avatar is generated based on the container name, however browsers tend to 
> cache pretty hard, so you might need to clear your cache/cookies before 
> hitting refresh to see a different avatar.

If you cannot see different avatars, use can use the command below to test with 
`curl` instead.  The command will `curl` the application 10 times and will list 
the container name.  You should see that it hits different pods:

```
for i in {1..10}; do curl -s $(minikube service container-info -n lab-07 --url) | egrep "<td>container-info"; done

          <td>container-info-6d9747978f-2x5r7</td>
          <td>container-info-6d9747978f-2x5r7</td>
          <td>container-info-6d9747978f-c9twz</td>
          <td>container-info-6d9747978f-c9twz</td>
          <td>container-info-6d9747978f-sqvtw</td>
          <td>container-info-6d9747978f-sqvtw</td>
          <td>container-info-6d9747978f-c9twz</td>
          <td>container-info-6d9747978f-sqvtw</td>
          <td>container-info-6d9747978f-2x5r7</td>
          <td>container-info-6d9747978f-sqvtw</td>
```

## Task 4: Cleaning up

Clean up the namespace for this lab:

```
kubectl delete ns lab-07

---

namespace "lab-07" deleted
```