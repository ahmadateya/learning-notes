# Helpful commands with kubectl

* To get the events of a speciefic pod
	* `kubectl get event --namespace=your-namespace --field-selector involvedObject.name=pod-name` 
* Bash on a sp container in a pod
	* `kubectl exec --stdin --tty <pod-name> --container <continer-name> bash` 
	
