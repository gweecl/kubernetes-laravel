# Laravel + GKE

## Setup 
1. Setup for GKE (https://cloud.google.com/kubernetes-engine/docs/quickstart)
2. Tutorial (https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)


## Deploy your images to GKE
```
// Set the PROJECT_ID environment variable in your shell
export PROJECT_ID="$(gcloud config get-value project -q)"

// Build images
// Set environment variables in your shell for images name or edit your images name in `docker-compose.yml`.
docker-compose build

// Push images to gcp container registry (https://cloud.google.com/container-registry/)
// The gcr.io prefix refers to Google Container Registry
docker push IMAGE_NAME:TAG

// Make sure cluster created
// OR create with gcloud cli
gcloud container clusters create hello-app
// Get authentication credentials for the cluster
gcloud container clusters get-credentials [CLUSTER_NAME]

// Deploy configMap
kubectl apply -f k8s/configurations/configmap-nginx.yaml

// Development
kubectl apply configmap configmap-laravel-env \
        --from-env-file=./docker/app/laravel.dev.env
// OR 
// Production
kubectl apply configmap configmap-laravel-env \
        --from-env-file=./docker/app/laravel.prod.env

// For HTTPS
// You can create self-signed certificates (https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl)
kubectl create secret tls secret-tls \
    --cert k8s/https/test-ingress-1.crt --key k8s/https/test-ingress-1.key

// Set environment variables in your shell and run the following command
cat k8s/deployments/deployment.yaml | \
sed 's/\${PROJECT_ID}'"/$PROJECT_ID/g" | \
sed 's/\${COMMIT_SHA}'"/$COMMIT_SHA/g" | \
kubectl apply -f -
// OR
// Edit your images name in ./k8s/deployments/deployment.yaml and run the following command
kubectl apply -f k8s/deployments/deployment.yaml 

// Services
kubectl apply -f k8s/services/service.yaml

// Expose and route your service with Ingress
kubectl apply -f k8s/ingress/ingress.yaml
```

> `kubectl apply -f DEPLOYMENT_FILE` may not trigger pods update if the manifest files / image_name:tag have no changes.  
>  Add or Remove `imagePullPolicy: "Always"` to K8s Deployment file and `kubectl apply` will force the image to be pulled and update the pods.


## Links

  * Configurations within the K8s - [Configure Access to Multiple Clusters or Namespaces](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).
  * Configurations with different Google accounts or GCP Projects - [Managing gcloud Configurations](https://cloud.google.com/sdk/docs/configurations)
  * Why use Ingress rather than native Load Balancer ? [A brief on the difference](https://stackoverflow.com/a/50285988/4778852). [Example 1](https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer) and [Example 2](https://kubernetes.io/docs/concepts/services-networking/ingress/#simple-fanout)
  * Run and expose multiple apps or services in single K8s cluster with Ingress - [Multiple name-based virtual hosting](https://kubernetes.io/docs/concepts/services-networking/ingress/#name-based-virtual-hosting)
  * I would suggest to use managed service like GCP Cloud SQL or AWS RDS for database. [Why ?](https://patrobinson.github.io/2017/12/16/should-i-run-a-database-in-kubernetes/)  
  Connect external services via [External Name](https://kubernetes.io/docs/concepts/services-networking/service/#externalname) and [Best Pratices](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-mapping-external-services).
  * Pods communications via [DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).
  * Secure your K8s cluster with a [default policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic) and limit access by [declare a policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/).
