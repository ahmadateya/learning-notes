# Kubernetes General Notes
* In general the POD in the node should have one container, and its not recommended to have more than one except if you have a helper container inside the pod and it called `sidecar container`

* creating a k8s NodePort service the strurcture will be like this
	* port: Port on node service listens in Kubernetes cluster internally
	* targetPort: We define container port here on which our application is running.
	* NodePort: Worker Node port on which we can access our application.
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-09-22%2008-41-08.jpg" width="300" height="300">

* [k8s cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [What is a Kubernetes Deployment?](https://www.vmware.com/topics/glossary/content/kubernetes-deployment#:~:text=A%20Kubernetes%20Deployment%20is%20used,earlier%20deployment%20version%20if%20necessary.)
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/kubernetes%20arch.png" width="300" height="300">
* 











## most of this notes came from my study over the [AWS EKS Kubernetes-Masterclass | DevOps, Microservices](https://www.udemy.com/course/aws-eks-kubernetes-masterclass-devops-microservices/) / Kuberbnetes Fundamentals
