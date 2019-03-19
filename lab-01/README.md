# Lab 01 - Minikube

For these labs we will use minikube as our Kubernetes environment. This is
going to run locally on your host.

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a 
single-node Kubernetes cluster inside a VM on your laptop for users looking to 
try out Kubernetes or develop with it day-to-day.

## Task 0: Installing VirtualBox

By default `minikube` uses `VirtualBox` to run its VM.  To install VirtualBox 
download the latest [https://download.virtualbox.org/virtualbox/6.0.4/VirtualBox-6.0.4-128413-OSX.dmg](package) 
and install it.


## Task 1: Installing minikube

The easiest way to install minikube is with [https://brew.sh/](homebrew). The 
following command will install minikube.

```
brew cask install minikube
```

## Task 2: Confirming minikube installation

To verify that the installation is succesfull we can try to get the version of
minikube with the following command :

```
minikube version

minikube version: v0.35.0
```

## Task 3: Running minikube

To start minikube run the `minikube start` command (it might take a couple of minutes 
before minikube has started):

```
minikube start

😄  minikube v0.35.0 on darwin (amd64)
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🏃  Re-using the currently running virtualbox VM for "minikube" ...
⌛  Waiting for SSH access ...
📶  "minikube" IP address is 192.168.99.102
🐳  Configuring Docker as the container runtime ...
✨  Preparing Kubernetes environment ...
🚜  Pulling images required by Kubernetes v1.13.4 ...
🔄  Relaunching Kubernetes v1.13.4 using kubeadm ...
⌛  Waiting for pods: apiserver proxy etcd scheduler controller addon-manager dns
📯  Updating kube-proxy configuration ...
🤔  Verifying component health ......
💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!
```

## Task 4: Checking minikube status

To verify that minikube is running correctly run the `minikube status` command:

```
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.102
```
