### TO-DO
* Add phpmyadmin and mysql DB for local environment :white_check_mark: 
* Try with GCP k8s engine. :white_check_mark: 
* Setup for HTTPS :white_check_mark: 
* try ingresss. y ? https://stackoverflow.com/a/50285988/4778852 :white_check_mark: 
* Try use config files to templatize build (UPDATE: helm is a good tools to do so)
* Build CI/CD and run test with GCP CloudBuild
* RBAC 


### ISSUES
* laravel log file permission issue
* Laravel environment file using volume mount method. Containers won't updated when configMap updated. Same thing goes to nginx.
* How to manage registry. nameing, multiple env etc. (variable subsituition not avaiable fpr k8s files) - https://medium.com/@adambarreiro/templating-your-kubernetes-files-5bb8097706f7