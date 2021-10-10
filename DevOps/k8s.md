# Kubernetes General Notes
* In general the POD in the node should have one container, and its not recommended to have more than one except if you have a helper container inside the pod and it called `sidecar container`

* there are 3 types of services in k8s you can recap them quickly and Ingress in this [video](https://www.youtube.com/watch?v=NPFbYpb0I7w&ab_channel=IBMTechnology) 
* creating a k8s NodePort service the strurcture will be like this
	* port: Port on node service listens in Kubernetes cluster internally
	* targetPort: We define container port here on which our application is running.
	* NodePort: Worker Node port on which we can access our application.
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-09-22%2008-41-08.jpg" width="350" height="300">

* [k8s cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [What is a Kubernetes Deployment?](https://www.vmware.com/topics/glossary/content/kubernetes-deployment#:~:text=A%20Kubernetes%20Deployment%20is%20used,earlier%20deployment%20version%20if%20necessary.)
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-09-10%2011-00-24.png" width="400" height="300">

* Deployment Strategies
	* in most cases you will use the rolling 

* To delete PV and PVC
	1. `kubectl patch pvc <pvc-name> -p '{"metadata":{"finalizers": null}}' --type=merge`
	2. `kubectl delete -f <file.yaml>`
	3. `kubectl delete pvc <pvc-name>` 
	* if the pvc stucks at `terminating` status you have to manually edit the pvc object and and remove `finalizers` object [link](https://github.com/kubernetes/kubernetes/issues/69697#issuecomment-447201890)
		* `kubectl edit pvc <pvc-name>` will open it in terminal and delete the `finalizers` object
	
* [What is Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
<img src="https://github.com/ahmadateya/learning-notes/blob/main/images/Screenshot%20from%202021-10-10%2010-45-44.png" width="550" height="250">







### most of this notes came from my study over the [AWS EKS Kubernetes-Masterclass | DevOps, Microservices](https://www.udemy.com/course/aws-eks-kubernetes-masterclass-devops-microservices/) / Kuberbnetes Fundamentals
