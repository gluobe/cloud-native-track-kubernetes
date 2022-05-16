# Lab 01 - Minikube

For these labs we will use [Minikube](https://github.com/kubernetes/minikube) as our Kubernetes environment. This is
going to run locally on a virtual machine on your laptop.

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a 
single-node Kubernetes cluster inside a VM on your laptop for users looking to 
try out Kubernetes or develop with it day-to-day.

## Task 0: Installing Minikube

If you already followed all steps in the prerequisites repo you can skip this step and move to `Task 1`.

If you did not, follow all steps in the [prerequisites repo](https://github.com/gluobe/cloud-native-track-prerequisites/tree/main/prereq-02-minikube)

## Task 1: Enable Minikube

```
minikube start
```

## Task 2: Test Minikube

Ensure that everything is working:

```
minikube status

---

minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## Task 3: Testing kubectl

To verify that `kubectl` has been configured correctly run the `kubectl get nodes` command:

```
kubectl get nodes

---

NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   197d   v1.19.4
```

> NOTE: if you do not see the `minikube` node you are most likely connected to the wrong cluster