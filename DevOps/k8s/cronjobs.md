# Notes About CronJobs in K8s


* Create a CronJob in k8s by following this [tutorial](https://medium.com/google-cloud/kubernetes-cron-jobs-455fdc32e81a). 
* CronJobs Should be idempotent read [this](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations).
* CronJobs creates a pod in every time it execute.
* Important thread for understanding CronJobs execution [CronJobs in Kubernetes - connect to existing Pod, execute script](https://stackoverflow.com/questions/41192053/cron-jobs-in-kubernetes-connect-to-existing-pod-execute-script).

* Alternatives to run cronJob without the k8s CronJob resource (but not tested) 
	* [Cron in a Pod](https://medium.com/airwalk/cron-in-a-pod-6fdf6676c623).
	* [Laravel: Docker + Cron Jobs](https://javicastilla.com/2020/08/21/laravel-docker-cron-jobs/).


* ### Running Laravel on Kubernetes
	* https://lorenzo.aiello.family/running-laravel-on-kubernetes/  
	* https://chris-vermeulen.com/laravel-in-kubernetes-part-9/  
	* [Good practice to run 'php artisan schedule:run' for Laravel in Kubernetes](https://stackoverflow.com/questions/66981054/good-practice-to-run-php-artisan-schedulerun-for-laravel-in-kubernetes)
	* https://www.jeffgeerling.com/blog/2019/running-php-artisan-schedulerun-laravel-kubernetes-cronjobs
