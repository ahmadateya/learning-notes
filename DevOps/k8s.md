# Installing and configuring minikube
* I folllowed this tutorial to download and install minkube [Install Minikube on Ubuntu 20.04/18.04 & Debian 10](https://computingforgeeks.com/how-to-install-minikube-on-ubuntu-debian-linux/) 


# Kubernetes General Notes
* In general the POD in the node should have one container, and its not recommended to have more than one except if you have a helper container inside the pod and it called `sidecar container`

* creating a k8s NodePort service the strurcture will be like this
	* port: Port on node service listens in Kubernetes cluster internally
	* targetPort: We define container port here on which our application is running.
	* NodePort: Worker Node port on which we can access our application.
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-09-22%2008-41-08.jpg" width="300" height="300">

* [k8s cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* 
