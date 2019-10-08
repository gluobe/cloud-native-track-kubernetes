# Lab 05 - YAML

Although relatively easy in concept, this lab is a very important one when 
working with Kubernetes.  In this lab we introduce YAML as a way to describe 
objects in Kubernetes.  So far we only used specific `kubectl` commands to 
create a limited set of objects.

If you are not very familiar with YAML check out the website below for a basic
introduction and its relevance for Kubernetes:
https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/

## Task 1: Creating objects using YAML

In the previous labs we have used `kubectl create namespace` to create a
namespace object in Kubernetes, instead we could (and probably should) have also
used YAML.  So what would this look like?

The YAML code to replace `kubectl create namespace lab-05` looks like
this:

```
apiVersion: v1
kind: Namespace
metadata:
  name: lab-05
```

To create the object in Kubernetes simply copy the above content into a file,
for example `lab-05-namespace.yml`.  After that simply issue the following
commmand:

```
kubectl apply -f lab-05-namespace.yml

---

namespace/lab-05 created
```

Check that the namespace has indeed been created:

```
kubectl get namespaces

---

NAME          STATUS   AGE
default       Active   22h
kube-public   Active   22h
kube-system   Active   22h
lab-05        Active   23s
```

## Task 2: Deleting objects using YAML

Deleting an object is even easier, only requirement is that you have the orignal
YAML file at your disposal:

```
kubectl delete -f lab-05-namespace.yml

---

namespace "lab-05" deleted
```

## Task 3: Multiple objects

When you have multiple objects you have different options:
* you can put multiple objects in the same YAML (using lists), we will cover 
this in a later lab
* you can have multiple YAML files inside a directory and `kubectl apply` the
directory (for example: `kubectl apply -f directory_full_of_yaml/`)

## Task 4: YAML and namespaces

When working with YAML you can use namespaces in different ways:
* as metadata in YAML
* as an option provided on the command line

As we deleted the namespace in a previous task we first need to recreate it:

```
kubectl apply -f lab-05-namespace.yml

---

namespace/lab-05 created
```

When working with metadata your YAML to create a single pod will look like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: lab-05
spec:
  containers:
  - name: nginx
    image: nginx
```

Notice the `namespace: lab-05` line in the metadata section.  This will force
the pod to be created inside the `lab-05` namespace.

Save the above content into `lab-05-pod-metadata-ns.yml` and apply it:

```
kubectl apply -f lab-05-pod-metadata-ns.yml

---

pod/nginx created
```

Or you can omit the namespace in the metadata and provide it at the command
line:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Save the above content into `lab-05-pod.yml` and apply it:
```
kubectl apply -f lab-05-pod.yml -n lab-05

---

pod/nginx unchanged
```

> NOTE: you will get the `pod/nginx` unchanged message as nothing has changed

Both options have their specific use-cases.

As some of you might wonder what happens when you both specify a namespace
inside the metadata as well as on the command line, so let us give this a try:

```
kubectl apply -f lab-05-pod-metadata-ns.yml -n default

---

error: the namespace from the provided object "lab-05" does not match the namespace "default". You must pass '--namespace=lab-05' to perform this operation.
```

As you can see, Kubernetes will not allow you to override the namespace that is
set in the yaml code.

## Task 5: Cleaning up

Clean up the namespace for this lab:

```
kubectl delete ns lab-05

---

namespace "lab-05" deleted
```
