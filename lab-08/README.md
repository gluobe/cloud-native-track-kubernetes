# Lab 08 - Config Maps & Secrets

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-08

---

namespace "lab-08" created
```

## Task 1: Creating a configmap

With a configmap we are able to easily pass environment variables to the 
deployment we are creating.

Create a file `container-info.env` with the following content:

```
# container-info.env

IMAGE_COLOR=green
```

This will pass an environment variable to the pods that we will create with our 
deployment. We are now able to create a configmap with this file.

```
kubectl create configmap container-info-env --from-env-file='./container-info.env' -n lab-08
```

Verify that the configmap has been created succesfully:

```
kubectl get configmap container-info-env -n lab-08

---

NAME                 DATA      AGE
container-info-env   1         59s
```

```
kubectl describe configmap container-info-env -n lab-08

---

Name:		container-info-env
Namespace:	lab-08
Labels:		<none>
Annotations:	<none>

Data
====
IMAGE_COLOR:
----
green
Events:	<none>
```

## Task 2: Adding the configmap in the deployment.

The deployment we create now looks a lot like the deployment we created in the 
previous labs.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: container-info
  labels:
    app: container-info
spec:
  replicas: 1
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
        image: gluobe/container-info:variable
        ports:
        - containerPort: 80
        envFrom:
          - configMapRef:
              name: container-info-env
```     

Copy the content above into a file `lab-08-deployment.yml`.

Now we create the new deployment with our environment variables.

```
kubectl apply -f lab-08-deployment.yml -n lab-08

---

deployment "container-info" created
```

We will expose the service again.

```
kubectl expose deployment container-info --type=NodePort --name=container-info -n lab-08

---

service "container-info" exposed
```

And connect to the service in the browser.

```
minikube service container-info -n lab-08
```

## Task 3: Updating an existing configmap

Configmaps make it possible to change configuration of a application (pod), 
without needing to completely rebuild a container image.  Let's try this.

Issue the following command, and replace the image color to `yellow` and save 
the  file (you will get a `configmap "container-info-env" edited` message when 
succesful):

```
kubectl edit configmap container-info-env -n lab-08

---

configmap "container-info-env" edited
```

Visit your application again to see if the color has indeed been changed:

```
minikube service container-info -n lab-08
```

You should notice that the color has NOT changed... Do not worry, this is 
expected behaviour. Before we see the change we need to restart the pod, for ease 
of use we  will do this by simply deleting the pod (Kubernetes we start a new 
one automatically).

First list (and copy) the name of the pod:

```
kubectl get pods -n lab-08

---

NAME                              READY     STATUS    RESTARTS   AGE
container-info-84dc4f8678-bbcbp   1/1       Running   0          18m
```

Now delete that pod:

```
kubectl delete pod container-info-84dc4f8678-bbcbp -n lab-08
```

If you are quick enough you will we temporary see two pods, one starting/running 
and one terminating/running:

```
NAME                              READY     STATUS        RESTARTS   AGE
container-info-84dc4f8678-2m9xm   1/1       Running       0          2s
container-info-84dc4f8678-bbcbp   0/1       Terminating   0          20m
```

Now visit your application again, this time you should see that the color has 
changed to yellow:

```
minikube service container-info -n lab-08
```

## Task 4 : Create a secret from literal

Secrets can be handled differently in a Kubernetes cluster. There are 3 ways to
handle these secrets. From `literal`, from a `file` and from a `YAML` file.

If you want to create a secret from `literal` it looks like this.

```
kubectl create secret generic lab-08-secret-literal --from-literal=password=N0T$oS3cREtP@SSw0rD -n lab-08

---

secret "lab-08-secret-literal" created
```

This secret can be listed by the command:

```
kubectl get secret -n lab-08

---

NAME                    TYPE                                  DATA      AGE
default-token-twfzc     kubernetes.io/service-account-token   3         8m
lab-08-secret-literal   Opaque                                1         25s
```

## Task 5: Creating a secret from a file

The second option we have is creating the secret from a file. First of all 
create the file we are going to use in the command.

```
echo -n "N0T$oS3cREtP@SSw0rD" > ./password.txt
```  

If this file is created we can use it to create the secret.

```
kubectl create secret generic lab-08-secret-file --from-file=./password.txt -n lab-08

---

secret "lab-08-secret-file" created
```

Verify that you secret has been created succesfully:

```
kubectl get secret -n lab-08

---

NAME                    TYPE                                  DATA      AGE
default-token-7sszf     kubernetes.io/service-account-token   3         4m
lab-08-secret-file      Opaque                                1         17s
lab-08-secret-literal   Opaque                                1         1m
```

## Task 6: Creating a secret from YAML

Our last option we have is to create the secret directly from a yaml. This is 
similar to how we create services, deployments,... and any other Kubernetes 
objects we already created in these labs.  First we need to encrypt our 
password.

> NOTE: as you will see by default secrets are simply base64 encoded, so not 
> really very secret

```
echo -n 'N0T$oS3cREtP@SSw0rD' | base64

 TjBUJG9TM2NSRXRQQFNTdzByRA==
```

This base64 encoded string we can you in the YAML file we will create.  Create 
a file `lab-08-secret-yaml.yml` with the content below:

```
apiVersion: v1
kind: Secret
metadata:
  name: lab-08-secret-yaml
type: Opaque
data:
  password: TjBUJG9TM2NSRXRQQFNTdzByRA==
```

In this yaml file we specify the output we generated from the base64 encode. At
this moment you are able to apply the yaml file and the secret will be created.

```
kubectl create -f ./lab-08-secret-yaml.yml -n lab-08

---

secret "lab-08-secret-yaml" created
```

Verify that you secret has been created succesfully:

```
kubectl get secret -n lab-08

---

NAME                    TYPE                                  DATA      AGE
default-token-7sszf     kubernetes.io/service-account-token   3         11m
lab-08-secret-file      Opaque                                1         6m
lab-08-secret-literal   Opaque                                1         7m
lab-08-secret-yaml      Opaque                                1         44s
```

## Task 7: Checking the content of a secret

We can describe a secret just like any other object in Kubernetes.

```
kubectl get secret lab-08-secret-yaml -o yaml -n lab-08
```

You will see the encoded password in the YAML. This can be decoded using the 
following command.

```
echo 'TjBUJG9TM2NSRXRQQFNTdzByRA==' | base64 --decode
```

This will output : `N0T$oS3cREtP@SSw0rD`

This is the password we used before in this lab.

## Task 8: Cleanup

Clean up the namespace for this lab:

```
kubectl delete ns lab-08

---

namespace "lab-08" deleted
```
