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

* Deployment Strategies
	* in most cases you will use the rolling 


* To delete PV and PVC
	1. `kubectl patch pvc PVC_NAME -p '{"metadata":{"finalizers": null}}' --type=merge`
	2. `kubectl delete -f <file.yaml>`
	3. `kubectl delete pvc <pvc-name>` 
	* if the pvc stucks at `terminating` status you have to manually edit the pvc object and and remove `finalizers` object [link](https://github.com/kubernetes/kubernetes/issues/69697#issuecomment-447201890)








### most of this notes came from my study over the [AWS EKS Kubernetes-Masterclass | DevOps, Microservices](https://www.udemy.com/course/aws-eks-kubernetes-masterclass-devops-microservices/) / Kuberbnetes Fundamentals
