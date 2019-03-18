# Lab 12 - Liveness / readiness probes

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-12

namespace "lab-12" created
```

## Task 1: Using readiness probe 

To test the readiness of a pod we are going to create the following file. Name 
the file `lab-12-probe-readiness.yml` and fill with the content below.

Pay special attention the the`readinessProbe` section.  This is the check that 
Kubernetes will perform continuously to see if a pod is ready or not.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: probe-readiness
  name: probe-readiness
spec:
  containers:
  - name: probe-readiness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 6000
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
      initialDelaySeconds: 5
      periodSeconds: 5
```

Apply the file in your namespace to create the `probe-readiness` pod.

```
kubectl apply -f lab-12-probe-readiness.yml -n lab-12

pod "probe-readiness" created
```

Now we need to check the state of the pod.

```
kubectl get pods -n lab-12

NAME              READY   STATUS    RESTARTS   AGE
probe-readiness   0/1     Running   0          10s
```

We see that the pod is running but not ready yet: `READY: 0/1`.

Before we are going to fix this readiness check we are going to create a 
service for this pod.

Create the file `lab-12-probes-service.yml` with this content.

```
kind: Service
apiVersion: v1
metadata:
  name: probes-service
spec:
  selector:
    app: probe-readiness
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Apply the file and check out the service.

```
kubectl apply -f lab-12-probes-service.yml -n lab-12

service "probes-service" created
```

When we describe the service at this moment we won't see an endpoint, this is 
expected.  Kubernetes will only create an endpoint for pods when they are in the 
ready state.

```
kubectl describe service probes-service -n lab-12

Name:                     probes-service
Namespace:                lab-10
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"probes-service","namespace":"lab-10"},"spec":{"ports":[{"port":80,"protocol":"...
Selector:                 app=probes
Type:                     NodePort
IP:                       10.105.221.153
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32748/TCP
Endpoints:
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

If we fix the readiness check on the pod (by creating the `/tmp/ready` file), 
our pod will become ready and will be added as an endpoint of the service. Let's 
create the file inside our pod:

```
kubectl exec probe-readiness touch /tmp/ready -n lab-12
```

Now you will see that the pod is ready.

```
kubectl get pods -n lab-12

NAME              READY   STATUS    RESTARTS   AGE
probe-readiness   1/1     Running   0          2m39s
```

And if we describe the service once again we will see that an endpoint has 
appeared.

```
kubectl describe service probes-service -n lab-12

Name:                     probes-service
Namespace:                lab-10
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"probes-service","namespace":"lab-10"},"spec":{"ports":[{"port":80,"protocol":"...
Selector:                 app=probes
Type:                     NodePort
IP:                       10.105.221.153
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32748/TCP
Endpoints:                172.17.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Delete the pod and service created in this task:

```
kubectl delete -f lab-12-probe-readiness.yml -n lab-12

pod "probe-readiness" deleted
```

```
kubectl delete -f lab-12-probes-service.yml -n lab-12

service "probes-service" deleted
```

## Task 2: Using liveness probe 

Now we will simulate a situation where the pod becomes unhealthy (in other words 
the `livenessProbe` will fail).  If a pods becomes unhealthy Kubernetes will try 
to fix this by deleting the pod (and starting a new pod).


To test the liveness of a pod we are going to create the following file. Name 
the file `lab-12-probe-liveness.yml` and fill with the content below.

Pay special attention the the`livenessProbe` section.  This is the check that 
Kubernetes will perform continuously to see if a pod is alive/healthy or not.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: probe-liveness
  name: probe-liveness
spec:
  containers:
  - name: probe-liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 6000
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

Apply the file in your namespace to create the `probe-liveness` pod.

```
kubectl apply -f lab-12-probe-liveness.yml -n lab-12

pod/probe-liveness created
```

Now we need to check the state of the pod.

```
kubectl get pods -n lab-12

NAME             READY   STATUS    RESTARTS   AGE
probe-liveness   1/1     Running   0          10s
```

To simulate the failure we are going to remove the `/tmp/healthy` file from our 
pod:

```
kubectl exec probe-liveness rm /tmp/healthy -n lab-12
```

Now wait +/- 1 minute and check the status of your pod again:

```
NAME             READY   STATUS    RESTARTS   AGE
probe-liveness   1/1     Running   1          2m10s
```

What you should see is that the restart count is now `1` (this was previously 
`0`). So we see that Kubernetes fixed our unhealthy pod by deleting it and 
starting a brand new one.

## Task 2: Cleanup

Clean up the namespace for this lab:

```
kubectl delete ns lab-12

namespace "lab-12" deleted
```