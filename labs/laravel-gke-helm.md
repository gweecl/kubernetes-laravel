# Laravel + GKE + Helm

## What is Helm and Why ?

* Building k8s cluster with declarative configuration files (YAML) is good. 
* But how you rolling updates / rollback ? `kubectl apply` for every configuration file. What ?!
* Using environment variables to templatize it ? Not supported.

There are discussion about these issues and suggestions on tools to templatize the manifest files:  
https://github.com/kubernetes/kubernetes/issues/23896#issuecomment-31354485  
  
So I decided to give it a try on [Helm](https://helm.sh/).  
  
Basically, Helm uses [Go templates](https://godoc.org/text/template) for templating
your manifest files (YAML). While ships with several built-in functions and other additonal features.  

Helm process those template with the values spcified in `values.yaml`. 
You can have multiple `values.yaml` file for different environments. 
Just like the concept in [Docker Compose](https://docs.docker.com/compose/extends/#compose-documentation).  
  
Understand [three big concepts](https://docs.helm.sh/using_helm/#three-big-concepts) in Helm.  


## Setup Helm

1. Install [Helm](https://docs.helm.sh/using_helm/#install-helm)

2. To configure Role-Based Access Control in GKE, you need to grant your user the ability to create role. [Find out more](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control).
    ```
    kubectl create clusterrolebinding cluster-admin-binding \
        --clusterrole cluster-admin --user [USER_ACCOUNT]
    ```
3. Configure Helm and Tiller. Tiller is a companion to the `helm` command that runs on your cluster.

    Give Tiller full access to cluster, which is not required.
    ```
    kubectl apply -f helm/rbac-config.yaml
    ```
    #### OR

    Restrict tiller access to a particular namespace.  
    In this project, we create a role  `default-tiller-role` which only have access to `default` namespace and bind it with `default-tiller-manager` service account.
    ```
    kubectl apply -f helm/default-tiller-role.yaml

    kubectl apply -f helm/default-tiller-rolebinding.yaml
    ```

    Find out more details [here](https://docs.helm.sh/using_helm/#tiller-and-role-based-access-control).
 
3. To begin working with Helm, run the `helm init` command, with the namespace spcified:
    ```
    helm init --tiller-namespace default --namespace default
    ```

    You can skip `--tiller-namespace default --namespace default` by set the tiller namespace in the shell
    ```
    export TILLER_NAMESPACE=default
    ```


## Manage K8s with Helm

1. Create a [Chart](https://docs.helm.sh/developing_charts/#using-helm-to-manage-charts).

2. Templatize your K8s manifest files in `templates` directory and configure `values.yaml` file.  
Find out more on [Devloper Guide](https://docs.helm.sh/chart_template_guide/#the-chart-template-developer-s-guide).

3. Install your chart.
    ```
    helm install --name CHART_NAME CHART_FOLDER_PATH
    ```

    Apply changes on chart.
    ```
    helm upgrade CHART_NAME CHART_FOLDER_PATH
    ```

    To override default `values.yaml`.
    ```
    // override values
    helm install -f myvalues.yaml --name CHART_NAME CHART_FOLDER_PATH
    helm upgrade -f myvalues.yaml -f override.yaml CHART_NAME CHART_FOLDER_PATH
    ```

    To check changes before apply.
    ```
    // To view in console
    helm upgrade --dry-run --debug -f myvalues.yaml -f override.yaml CHART_NAME CHART_FOLDER_PATH
    // To export as file
    helm upgrade --dry-run --debug -f myvalues.yaml -f override.yaml CHART_NAME CHART_FOLDER_PATH > example.yaml
    ```
5. Take a look for [example](/helm-laravel-chart) in this project.
    * use `--set-file` which allow to reach external files outside chart directory. [Find out more](https://github.com/helm/helm/issues/4026).
    * use `--set` to override the value in `values.yaml`.
    ```
    helm install --tiller-namespace default --namespace default \
    --set-file laravel.env=docker/app/laravel.prod.env \
    --set image.nginx.repository=IMAGE_NAME:TAG \
    --set image.phpLaravel.repository=IMAGE_NAME:TAG \
    --name CHART_NAME ./helm-laravel-chart
    ```

    To configure HTTPS for Ingress.
    ```
    helm install --tiller-namespace default --namespace default \
    --set-file ingress.secrets[0].certificate=YOUR_CERT_FILE \
    --set-file ingress.secrets[0].key=YOUR_KEY_FILE \
    --set-file laravel.env=docker/app/laravel.prod.env \
    --set image.nginx.repository=IMAGE_NAME:TAG \
    --set image.phpLaravel.repository=IMAGE_NAME:TAG \
    CHART_NAME ./helm-laravel-chart
    ```

    > `kubectl apply -f DEPLOYMENT_FILE` may not trigger pods update if the manifest files / image_name:tag have no changes.  
    >  I supposed its not the same with Helm. But its not. :disappointed:  
    >  Add or Remove `imagePullPolicy: "Always"` to K8s Deployment file and `helm upgrade` will force the image to be pulled and update the pods.  
  
5. To verify the app is installed:
    ```
    // DOMAIN_NAME in this project: my-laravel-app.com
    curl -H "host: DOMAIN_NAME" http://LOAD_BALANCER_IP
    ```


## Links

* [Configure RBAC In Your Kubernetes Cluster](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/)
