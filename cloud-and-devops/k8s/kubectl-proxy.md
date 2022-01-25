## You can open the k8s Dashboard in you local machine by using `kubectl-proxy --port=8999`
* To log in use the token you get from this command `kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')`


References
* https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/
