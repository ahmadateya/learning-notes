# Some helper Tools

* ### [minikube](https://github.com/ahmadateya/learning-notes/blob/main/DevOps/k8s/minikube.md).
* ###  Faster way to switch between clusters and namespaces in kubectl [kubectx + kubens](https://github.com/ahmetb/kubectx/) install it from [here](https://gist.github.com/argentinaluiz/a5dc8b1b58995bbbe98e37d9936ea436#install-kubectx-and-kubens).
	* you may need to install [krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/).

	* #### How to use
		```sh
		# switch to anoter cluster that's in kubeconfig
		$ kubectx minikube
		Switched to context "minikube".

		# switch back to previous cluster
		$ kubectx -
		Switched to context "oregon".

		# create an alias for the context
		$ kubectx dublin=gke_ahmetb_europe-west1-b_dublin
		Context "dublin" set.
		Aliased "gke_ahmetb_europe-west1-b_dublin" as "dublin".

		# change the active namespace on kubectl
		$ kubens kube-system
		Context "test" set.
		Active namespace is "kube-system".

		# go back to the previous namespace
		$ kubens -
		Context "test" set.
		Active namespace is "default".
		```
	

