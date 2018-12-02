# Laravel + GKE

## Setup and Tutorial
1. Setup for GKE (https://cloud.google.com/kubernetes-engine/docs/quickstart)
2. Tutorial (https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)


## Deploy Laravel to GKE

1. Set the the environment variables in your shell. Build and push the images to [Google Container Registry](https://cloud.google.com/container-registry/) with a name like `gcr.io/GCLOUD_PROJECT_ID/IMAGE_NAME:TAG`.
    ```
    export PROJECT_ID="$(gcloud config get-value project -q)"
    export COMMIT_SHA="v1.0.0"

    // Build images using `docker-compose.yml`
    docker-compose build

    // Push images to gcp container registry
    docker push IMAGE_NAME:TAG
    ```
2. Create a k8s cluster.
    ```
    gcloud container clusters create [CLUSTER_NAME]

    // Get authentication credentials for the cluster
    gcloud container clusters get-credentials [CLUSTER_NAME]

    // Setting a default compute zone (https://cloud.google.com/compute/docs/regions-zones/#available)
    gcloud config set compute/zone [COMPUTE_ZONE]
    ```

3. Configure your containers using [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/).
    ```
    // Nginx configuration
    kubectl apply -f k8s/configurations/configmap-nginx.yaml

    // Laravel environment
    // Files for different environments are located in ./docker/app directory
    kubectl apply configmap configmap-laravel-env \
        --from-env-file=./docker/app/laravel.dev.env

4. (Optional) For HTTPS. You can create a self-signed certificates. [Learn more](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl)
    ```
    kubectl create secret tls secret-tls \
        --cert [CERTIFICATE_FILE] --key [KEY_FILE]

5. Deploy kurbenetes workloads and services like [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) and [Services](https://kubernetes.io/docs/concepts/services-networking/service/) from manifest files.
    ```
    // Deployments
    kubectl apply -f k8s/deployments/deployment.yaml 

    // Services
    kubectl apply -f k8s/services/service.yaml

    // Expose and route your service with Ingress
    kubectl apply -f k8s/ingress/ingress.yaml
    ```

    > `kubectl apply -f DEPLOYMENT_FILE` may not trigger pods update if the manifest files / image_name:tag have no changes.  
    >  Add or Remove `imagePullPolicy: "Always"` to K8s Deployment file and `kubectl apply` will force the image to be pulled and update the pods.  


5. To verify the app is installed:
    ```
    // DOMAIN_NAME in this project: my-laravel-app.com
    curl -H "host: DOMAIN_NAME" http://LOAD_BALANCER_IP
    ```

## Links

  * Configurations within the K8s - [Configure Access to Multiple Clusters or Namespaces](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).
  * Configurations with different Google accounts or GCP Projects - [Managing gcloud Configurations](https://cloud.google.com/sdk/docs/configurations)
  * Why use Ingress rather than native Load Balancer ? [A brief on the difference](https://stackoverflow.com/a/50285988/4778852). [Example 1](https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer) and [Example 2](https://kubernetes.io/docs/concepts/services-networking/ingress/#simple-fanout)
  * Run and expose multiple apps or services in single K8s cluster with Ingress - [Multiple name-based virtual hosting](https://kubernetes.io/docs/concepts/services-networking/ingress/#name-based-virtual-hosting)
  * I would suggest to use managed service like GCP Cloud SQL or AWS RDS for database. [Why ?](https://patrobinson.github.io/2017/12/16/should-i-run-a-database-in-kubernetes/)  
  Connect external services via [External Name](https://kubernetes.io/docs/concepts/services-networking/service/#externalname) and [Best Pratices](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-mapping-external-services).
  * Pods or Services communications via [DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). So 
  * Secure your K8s cluster with a [default policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic) and limit access by [declare a policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/).
