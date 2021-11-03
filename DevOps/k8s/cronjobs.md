# Notes About CronJobs in K8s


* Create a CronJob in k8s by following this [tutorial](https://medium.com/google-cloud/kubernetes-cron-jobs-455fdc32e81a). 
* CronJobs Should be idempotent read [this](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations).
* CronJobs creates a pod in every time it execute.

* Nice Alternative to k8s cronJob (but not tested) [here](https://javicastilla.com/2020/08/21/laravel-docker-cron-jobs/).
