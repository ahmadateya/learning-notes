# Notes about Health checking in k8s


* health checks called probes in k8s
	* probes types:
		1. **liveness:** The kubelet uses liveness probes to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.
		2. **readiness:** The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.
		3. **startup:** The kubelet uses startup probes to know when a container application has started. If such a probe is configured, it disables liveness and readiness checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running

* [Advanced Health Check Patterns in Kubernetes](https://ahmet.im/blog/advanced-kubernetes-health-checks/) this article answers many questions for using the probes in:
	* Secondary Health Port
	* Sidecar health server
	* and many others







## Resources
* [kube by example](https://kubebyexample.com/en/concept/health-checks)
* [Probe v1 core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#probe-v1-core)
* [k8s docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
* [Kubernetes best practices: Setting up health checks with readiness and liveness probes by Google Cloud](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-setting-up-health-checks-with-readiness-and-liveness-probes)
* [Advanced Health Check Patterns in Kubernetes](https://ahmet.im/blog/advanced-kubernetes-health-checks/) 
