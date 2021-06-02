# Lab 01 - Minikube

For these labs we will use [Minikube](https://github.com/kubernetes/minikube) as our Kubernetes environment. This is
going to run locally on a virtual machine on your laptop.

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a 
single-node Kubernetes cluster inside a VM on your laptop for users looking to 
try out Kubernetes or develop with it day-to-day.

## Task 1: Installing Minikube

The easiest way to install Minikube is by using [homebrew](https://brew.sh/). The 
following command will install Minikube.

```
brew install minikube

---

==> Satisfying dependencies
All Formula dependencies satisfied.
==> Downloading https://storage.googleapis.com/minikube/releases/v1.4.0/minikube-darwin-amd64
######################################################################## 100.0%
==> Verifying SHA-256 checksum for Cask 'minikube'.
==> Installing Cask minikube
==> Linking Binary 'minikube-darwin-amd64' to '/usr/local/bin/minikube'.
🍺  minikube was successfully installed!
```

## Task 2: Confirming minikube installation

To verify that the installation is succesfull we can try to get the version of
Minikube with the following command :

```
minikube version

---

minikube version: v1.4.0
commit: 7969c25a98a018b94ea87d949350f3271e9d64b6
```

## Task 3: Running Minikube

To start Minikube run the `minikube start` command (it might take a couple of 
minutes before Minikube has started):

```
minikube start

---

😄  minikube v1.4.0 on Darwin 10.14.6
👍  Upgrading from Kubernetes 1.14.2 to 1.16.0
💿  Downloading VM boot image ...
    > minikube-v1.4.0.iso.sha256: 65 B / 65 B [--------------] 100.00% ? p/s 0s
    > minikube-v1.4.0.iso: 135.73 MiB / 135.73 MiB [-] 100.00% 17.28 MiB p/s 8s
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Starting existing virtualbox VM for "minikube" ...
⌛  Waiting for the host to be provisioned ...
🐳  Preparing Kubernetes v1.16.0 on Docker 18.09.6 ...
💾  Downloading kubelet v1.16.0
💾  Downloading kubeadm v1.16.0
🚜  Pulling images ...
🔄  Relaunching Kubernetes using kubeadm ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```

## Task 4: Checking Minikube status

To verify that minikube is running correctly run the `minikube status` command:

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

## Task 5: Checking kubectl status

To verify that `kubectl` has been configured correctly run the `kubectl get nodes` command:

```
kubectl get nodes

---

NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   197d   v1.19.4
```

> NOTE: if you do not see the `minikube` node you are most likely connected to the wrong cluster
