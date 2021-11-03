# Notes About CronJobs in K8s


* Create a CronJob in k8s by following this [tutorial](https://medium.com/google-cloud/kubernetes-cron-jobs-455fdc32e81a). 
* CronJobs Should be idempotent read [this](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations).
* CronJobs creates a pod in every time it execute.
* Important thread for understanding CronJobs execution [CronJobs in Kubernetes - connect to existing Pod, execute script](https://stackoverflow.com/questions/41192053/cron-jobs-in-kubernetes-connect-to-existing-pod-execute-script).

* Alternatives to run cronJob without the k8s CronJob resource (but not tested) 
	* [Cron in a Pod](https://medium.com/airwalk/cron-in-a-pod-6fdf6676c623).
	* [Laravel: Docker + Cron Jobs](https://javicastilla.com/2020/08/21/laravel-docker-cron-jobs/).
