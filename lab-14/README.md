# Lab 14 - Dashboard

The dashboard serves different functions, primarily it is used to visualize your 
Kubernetes cluster, however it can also be used to deploy new objects into your 
cluster.  In this small lab we will explore both functions.

## Task 1: Opening the dashboard

Opening the Kubernetes dashboard with minikube is very easy, simply run the 
following command and the dashboard will open automatically in your browser: 

```
minikube dashboard

🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:57922/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

## Task 2: Creating a namespace

We will now create a namespace using the dashboard UI.

Click the `+ CREATE` link at the top right corner of the dashboard.  Now select 
the `CREATE FROM TEXT INPUT` tab.  Copy the content below and paste it into the 
form:

```
apiVersion: v1
kind: Namespace
metadata:
  name: lab-14
```

Finally click the `UPLOAD` button.

In the left menu select the `Namespaces` item and verify that your namespace has 
been created succesfully.

## Task 3: Deploying an app

To deploy an app using the dashboard UI click the `+ CREATE` link at the top 
right corner of the dashboard.  Now select the `CREATE AN APP` tab.

Fill in the following details:

* `App name`: container-info
* `Container image`: gluobe/container-info:blue
* `Service`: External
* `Port`: 8888
* `Target port`: 80

Click the `SHOW ADVANCED OPTIONS` link and add some additional details:

* `Namespace`: lab-14

Click the `DEPLOY` button at the bottom.

## Task 4: Opening your app

To open you app, go back to your terminal and enter the following command:

```
minikube service container-info -n lab-14

🎉  Opening kubernetes service lab-14/container-info in default browser...
```

You should see a familiar website.

## Task 5: Deleting a namespace

While you can easily add namespaces (or other objects) from the dashboard, you 
cannot delete objects from there.  So delete the namespace, head back over to 
your terminal and run the following command:

```
kubectl delete ns lab-14

namespace "lab-14" deleted
```