# Lab 09 - Persistent Volumes

Persistent volumes can be used to store stateful/persistent data, this means
that when a pods is replaced its successor will still have the data available.
This can be used when you want to run persistent databases in pods and/or when
you want to share data between different pods.

In this lab we will make a persistent volume that we fill with images, those
images will then be served by our application (pods).

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-09

---

namespace "lab-09" created
```

## Task 1: Generating some stateful/persistent data

First we will need to ssh into the minikube instance.

```
minikube ssh
```

When you are in the minikube instance make sure that you are root. Otherwise you
might not be have permission to create the required directories and/or files:

```
sudo su -
```

Now we need to create a directory to store our images.

```
mkdir -p /mnt/data/images
```

Let's download a couple of images into our persistent images directory:

```
curl -so /mnt/data/images/meme1.jpeg https://raw.githubusercontent.com/gluobe/cloud-native-track-kubernetes/master/lab-09/images/meme1.jpeg
curl -so /mnt/data/images/meme2.jpeg https://raw.githubusercontent.com/gluobe/cloud-native-track-kubernetes/master/lab-09/images/meme2.jpeg
curl -so /mnt/data/images/meme3.jpeg https://raw.githubusercontent.com/gluobe/cloud-native-track-kubernetes/master/lab-09/images/meme3.jpeg
curl -so /mnt/data/images/meme4.jpeg https://raw.githubusercontent.com/gluobe/cloud-native-track-kubernetes/master/lab-09/images/meme4.jpeg
curl -so /mnt/data/images/meme5.jpeg https://raw.githubusercontent.com/gluobe/cloud-native-track-kubernetes/master/lab-09/images/meme5.jpeg
```

Now `exit` the minikube instance.

```
exit

exit
```

## Task 2: Create a persistent volume

In this task we are going to create the persistent volume. This will put the
volume in a `Available` state. It's not yet bound to a persistent volume claim
at this point.  Keep in mind that a persistent volume is not namespaced!

Create a file `lab-09-pv.yml` with the following content.

```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: lab-09-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadOnlyMany
  hostPath:
    path: "/mnt/data/images"
```

You can now create the persistent volume with the kubectl command. It will look
like this.

```
kubectl apply -f lab-09-pv.yml

---

persistentvolume "lab-09-volume" created
```

At this point the persistent volume is created. Be sure that it is by using the
get command.

```
kubectl get pv

---

NAME            CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
lab-09-volume   1Gi        ROX           Retain          Available             manual                   23s
```

## Task 3: Claim a persistent Volume

Now we need to create the claim for this persistent volume. This way we can bind
the persistent volume claim to a deployment. First we need to create the file
`lab-09-pvc.yml` with the following content.

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: lab-09-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 1Gi
```

And apply it with the following command.

```
kubectl apply -f lab-09-pvc.yml -n lab-09

---

persistentvolumeclaim/lab-09-claim created
```

You can confirm that it is created with the following command.

```
kubectl get pvc -n lab-09

---

NAME           STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
lab-09-claim   Bound    lab-09-volume   1Gi        ROX            manual         28s
```

## Task 4: Mounting a persistent volume in pods

Now we are able to use the PersistentVolumeClaim in a pod. The following
steps will show you how you can do this.

First we need to create a file that describes our pod. This will be
`lab-09-deployment.yml` with the following content.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meme-persistent
  labels:
    app: meme-persistent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: meme-persistent
  template:
    metadata:
      labels:
        app: meme-persistent
    spec:
      volumes:
      - name: lab-09-volume
        persistentVolumeClaim:
          claimName: lab-09-claim
      containers:
      - name: meme-persistent
        image: gluobe/meme-persistent:v1
        ports:
        - containerPort: 80
        volumeMounts:
          - mountPath: "/var/www/html/images"
            name: lab-09-volume
```

Apply the file with the kubectl command:

```
kubectl create -f lab-09-deployment.yml -n lab-09

---

deployment.apps/meme-persistent created
```

Be sure that the pods are running.

```
kubectl get pods -n lab-09

---

meme-persistent-84cdc97446-dc2v4   1/1     Running   0          31s
meme-persistent-84cdc97446-dh59c   1/1     Running   0          31s
meme-persistent-84cdc97446-tq8h8   1/1     Running   0          31s
```

Create a service to access the application, create a file `lab-09-service.yml`
with the content below:

```
kind: Service
apiVersion: v1
metadata:
  name: meme-persistent
spec:
  selector:
    app: meme-persistent
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Apply the file:

```
kubectl apply -f lab-09-service.yml -n lab-09

---

service/meme-persistent created
```

Check the application:

```
minikube service meme-persistent -n lab-09
```

When you refresh the page you should see that again the avatar changes (there
are three pods), but the images remain the same (the PHP script will simply
show all the images from the persistent volume directory).

You can download one additional image and this should instantly be available on
all the pods:

```
minikube ssh
```

Make sure you are `root`:

```
sudo su -
```

Download an additional image:

```
curl -so /mnt/data/images/kubernetes.jpeg https://raw.githubusercontent.com/gluobe/cloud-native-track-kubernetes/master/lab-09/images/kubernetes.png
```

Refresh your browser to see the new image.

## Task 5: Cleanup

Clean up the namespace for this lab:

```
kubectl delete ns lab-09

---

namespace "lab-09" deleted
```

> NOTE: we have now deleted our namespace, but as our persistent volume is not 
> namespaced it still exists.  So even if you deleted your application your data 
> will still be available.  If you are sure you no longer need your data follow 
> the steps below to clean up your persistent volume (and its data)

To delete the persistent volume object (this will not delete any data!):

```
kubectl delete -f lab-09-pv.yml
```

To delete the actual data:

```
minikube ssh

sudo su -

rm -rf /mnt/data/
```
