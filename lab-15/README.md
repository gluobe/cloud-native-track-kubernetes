# Lab 15 - Resource limiting

## 1 - Limiting a pod's resources

When you describe a Pod, you can optionally specify how much CPU and memory 
(RAM) each Container needs. When containers have resource requests specified, 
the scheduler can make better decisions about which nodes to place Pods on. And 
when Containers have their limits specified, contention for resources on a node 
can be handled in a specified manner.

### Requests and Limits

Requests are what the container is guaranteed to get. A pod will always be 
scheduled to a node that has the resources available to meet the request. 
Limits, on the other hand, make sure a container never goes above a certain 
value. The container is only allowed to go up to the limit, and then it is 
restricted.

### Resource types

CPU and memory are each a resource type. A resource type has a base unit. CPU is 
specified in units of cores, and memory is specified in units of bytes.

CPU and memory are collectively referred to as compute resources, or just 
resources. Compute resources are measurable quantities that can be requested, 
allocated, and consumed. They are distinct from API resources. API resources, 
such as Pods and Services are objects that can be read and modified through the 
Kubernetes API server.

#### Cpu

Fractional requests are allowed. A Container with spec.containers[].resources.requests.cpu 
of 0.5 is guaranteed half as much CPU as one that asks for 1 CPU. The expression 
0.1 is equivalent to the expression 100m, which can be read as “one hundred 
millicpu”. Some people say “one hundred millicores”, and this is understood to 
mean the same thing.

So basicly cpu: "500m" means 0.5 cpu, and cpu: "2000m" means 2 cpu's.

#### Memory

Limits and requests for memory are measured in bytes. You can express memory as 
a plain integer or as a fixed-point integer using one of these suffixes: E, P, 
T, G, M, K. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, 
Ki. For example, the following represent roughly the same value:

```
128974848, 129e6, 129M, 123Mi
```

### Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-15

---

namespace/lab-15 created
```

### Task 1: Describing a pod with resource limits

Create a file named `lab-15-limited-pod.yml` with the following content.

```
apiVersion: v1
kind: Pod
metadata:
  name: container-info
spec:
  containers:
  - name: container-info
    image: gluobe/container-info:blue
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Task 2: Apply the the file to the Kubernetes cluster:

```
kubectl apply -f lab-15-limited-pod.yml -n lab-15

---

pod/container-info created
```

Now the container-info container will be scheduled on a node that has 64Mi of 
memory available and 0.25 cpu's. It’s when it comes to CPU limits that things 
get interesting. CPU is considered a “compressible” resource. If your app starts 
hitting your CPU limits, Kubernetes starts throttling your container. This means 
the CPU will be artificially restricted, giving your app potentially worse 
performance! However, it won’t be terminated or evicted. Unlike CPU resources, 
memory cannot be compressed. Because there is no way to throttle memory usage, 
if a container goes past its memory limit it will be terminated. If your pod is 
managed by a Deployment, StatefulSet, DaemonSet, or another type of controller, 
then the controller spins up a replacement.

## 2 - Limiting a namespace's resources

Because sometimes same teams work on the same kubernetes cluster but in a 
different namespace, it is important to limit people from taking more than their 
fair share of the cluster.

To prevent these scenarios, you can set up ResourceQuotas and LimitRanges at the 
Namespace level.

### ResourceQuotas

After creating Namespaces, you can lock them down using ResourceQuotas. 
ResourceQuotas are very powerful, but let’s just look at how you can use them to 
restrict CPU and Memory resource usage.

### Task 3: Describing a ResourceQuota

Create the following file named `lab-15-resource-quota.yml`.

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: "demo"
spec:
  hard:
    requests.cpu: "500m"
    requests.memory: "100Mi"
    limits.cpu: "700m"
    limits.memory: "500Mi"
```

### Task 4: Apply this file to the cluster

```
kubectl apply -f lab-15-resource-quota.yml -n lab-15
```

This will have created a ResourceQuota which on the namespace lab-15. If we try 
and create a pod which exceeds the limits from the ResourceQuota we will get an 
error. Edit the file from our limited pod into this file:

```
apiVersion: v1
kind: Pod
metadata:
  name: container-info2
spec:
  containers:
  - name: container-info
    image: gluobe/container-info:blue
    resources:
      requests:
        memory: "600Mi"
        cpu: "1m"
      limits:
        memory: "1000Mi"
        cpu: "1.5m"

```

### Task 5: Apply this pod to the namespace:

```
kubectl apply -f lab-15-limited-pod.yml -n lab-15

---

Error from server (Forbidden): error when creating "lab15-limited-pod.yml": pods "container-info" is forbidden: exceeded quota: demo, requested: limits.memory=1000Mi,requests.memory=600Mi, used: limits.memory=0,requests.memory=0, limited: limits.memory=500Mi,requests.memory=100Mi
```

### Task 6: Remove ResourceQuota from namespace

```
kubectl delete resourcequota demo -n lab-15

---

resourcequota "demo" deleted
```

### LimitRange

You can also create a LimitRange in your namespace. Unlike a Quota, which looks 
at the namespace as a whole, a LimitRange applies to an individual container. 
This can help prevent people from creating super tiny or super large containers 
inside the namespace.

### Task 7: Describing a LimitRange

Create a file named `lab-15-limitrange.yml` with the following content:

```
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-mem-cpu-per-container
spec:
  limits:
  - max:
      cpu: "800m"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "99Mi"
    default:
      cpu: "700m"
      memory: "900Mi"
    defaultRequest:
      cpu: "110m"
      memory: "111Mi"
    type: Container

```

### Task 8: Apply this limitRange to our namespace:

```
kubectl apply -f lab-15-limitrange.yml -n lab-15

---

limitrange/limit-mem-cpu-per-container created
```

As you can see the yaml file has 4 sections:

The ***max section*** will set up the ***maximum limits*** that a container in 
a Pod can set. The default section cannot be higher than this value. Likewise, 
limits set on a container cannot be higher than this value.

The ***min section*** sets up the ***minimum Requests*** that a container in a 
Pod can set. The defaultRequest section cannot be lower than this value. 
Likewise, requests set on a container cannot be lower than this value either.

The ***default section*** sets up the ***default limits*** for a container in a 
pod. If you set these values in the limitRange, any containers that don’t 
explicitly set these themselves will get assigned the default values.

The ***defaultRequest section*** sets up the ***default requests*** for a 
container in a pod. If you set these values in the limitRange, any containers 
that don’t explicitly set these themselves will get assigned the default values.

### Task 9: Try and create a pod that exceeds these limits we will get an error.

```
kubectl apply -f lab-15-limited-pod.yml -n lab-15

---

Error from server (Forbidden): error when creating "lab15-limited-pod.yml": pods "container-info" is forbidden: minimum cpu usage per Container is 100m, but request is 1m

```

### Task 10: Clean up

```
kubectl delete namespace lab-15

---

namespace "lab-15" deleted
```
