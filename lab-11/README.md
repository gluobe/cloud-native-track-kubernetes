# Lab 11 - Blue / green deployments

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-11

---

namespace "lab-11" created
```

## Task 1: Blue / green deployment

For this task we are creating 2 versions of 1 deployment. This means that we
can use the service to point to either of the deployments. This can be used for
example if you want to update your application. We are going to use the
container-info application again to demo this feature.

First we will create a blue deployment. This is the first and initial version
of our application.

Create a file `lab-11-deployment-blue.yml` with the following content.

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
```

As you can see we have added a new label called `version` with this label we can
select the correct deployment in the service we are creating in this task.

```
kubectl apply -f lab-11-deployment-blue.yml -n lab-11

---

deployment.apps/container-info-blue created
```

Now create the file `lab-11-service-blue-green.yml` with the following content.

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
    version: "blue"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Notice that in the `spec:selector:version` we chose for `blue` this specifies
the version of the deployment we are going to expose.

```
kubectl apply -f lab-11-service-blue-green.yml -n lab-11

---

service/container-info created
```

Confirm that the application is running by doing to following command.

```
minikube service container-info -n lab-11
```

You will see that our `blue` version of the deployment is now running on our
minikube instance.

After you exposed the `blue` version we are going to create a `green` version
of the application. This is done by creating a file
`lab-11-deployment-green.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info-green
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
        version: "green"
    spec:
      containers:
      - name: container-info
        image: gluobe/container-info:green
        ports:
        - containerPort: 80
```

We changed some parameters from `blue` to `green`, notice that the image changed
as well. We are now deploying a `green` version.

```
kubectl apply -f lab-11-deployment-green.yml -n lab-11

---

deployment.apps/container-info-green created
```

If you list the deployments in your namespace you will see that we now have 2
deployments.

```
kubectl get deployment -n lab-11

---

NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
container-info-blue    3         3         3            3           12m
container-info-green   3         3         3            3           12m
```

A `blue` and a `green` version. Remember that our service is still pointing at
the `blue` version. In the next step we will update the service in a way that it
will point to the `green` version of our deployment.

Edit the `lab-11-service-blue-green.yml` file with the content below. You can 
also just clear the file and copy the following content.

```
kind: Service
apiVersion: v1
metadata:
  name: container-info
spec:
  selector:
    app: container-info
    version: "green"
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

The only thing that is updated in this file is basically the `version` this is
set to `green` now.

```
kubectl apply -f lab-11-service-blue-green.yml -n lab-11

---

service/container-info configured
```

Now you can run the service command again to check your newly updated version of
the deployment.

```
minikube service container-info -n lab-11
```

We can now delete the blue deployment:

```
kubectl delete -f lab-11-deployment-blue.yml -n lab-11

---

deployment.apps "container-info-blue" deleted
```

You now only have one deployment running, the `green`:

```
kubectl get deployment -n lab-11

---

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
container-info-green   3/3     3            3           3m31s
```

Congratulations, you have successfully performed you first `blue / green` 
deployment on `minikube`!

## Task 2: Cleanup

Clean up the namespace for this lab:

```
kubectl delete ns lab-11

---

namespace "lab-11" deleted
```