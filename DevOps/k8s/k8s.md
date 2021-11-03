# Kubernetes General Notes
* In general the POD in the node should have one container, and its not recommended to have more than one except if you have a helper container inside the pod and it called `sidecar container`


* [kubectl apply vs kubectl create?](https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create).
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

* To access and get inside a POD use `kubectl exec --stdin --tty <pod-name> /bin/bash`
* Unlike deployments and services in Kubernetes, you can't change the same Job configuration file and reapply it at once. When you make changes in the Job configuration file, you must delete the previous Job from the cluster before you apply it.

* Notes About CronJobs [here](https://github.com/ahmadateya/learning-notes/blob/main/DevOps/k8s/cronjobs.md).
* Faster way to switch between clusters and namespaces in kubectl [kubectx](https://github.com/ahmetb/kubectx/).
	* you may need to install [krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/). 


## Resources
* https://github.com/kubernetes-sigs/external-dns/blob/master/docs/faq.md
* https://aws.amazon.com/premiumsupport/knowledge-center/create-alb-auto-register/
* https://medium.com/@houseparty/introducing-targetgroupcontroller-a-way-to-more-efficiently-run-your-kubernetes-services-27b3dbc84e50
* [AWS EKS Kubernetes-Masterclass | DevOps, Microservices](https://www.udemy.com/course/aws-eks-kubernetes-masterclass-devops-microservices/)  / Kuberbnetes Fundamentals
* [Good k8s Series Examples](https://github.com/jonbcampos/kubernetes-series).

