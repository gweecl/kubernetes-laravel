# Laravel + Kubernetes (Minikube)

## Setup

1. [Install Minikube and kubectl](https://kubernetes.io/docs/tasks/tools/install-minikube/). Find out more details about Minikube setup [here](https://kubernetes.io/docs/setup/minikube/).
2. Start minikube
    ```
    minikube start
    ```

    [K8s Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) available with Minikube
    ```
    minikube dashboard
    ```

## Deploy your images to minikube
> To deploy local docker images (without pushed to registry or repo), you have to send the images to minikube. 
Find out more details [here](https://stackoverflow.com/questions/49898535/kubernetes-fails-to-run-a-docker-image-build-locally) and [here](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615).
> ```
> docker save YOUR_IMAGE_NAME:YOUR_IMAGE_TAG | (eval $(minikube docker-env) && docker load) 
> ```

1. Deploy kurbenetes workloads and services from manifest files like [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) and [Services](https://kubernetes.io/docs/concepts/services-networking/service/).
    ```
    // These are example for this project

    kubectl apply -f k8s/configurations/configmap-nginx.yaml

    // Set laravel environment for app container.
    // Files for different environments are located in ./docker/app directory
    kubectl apply configmap configmap-laravel-env \
        --from-env-file=./docker/app/laravel.dev.env

    // Set environment variables in your shell and run the following command
    cat k8s/deployments/deployment.yaml | \
    sed 's/\${PROJECT_ID}'"/$PROJECT_ID/g" | \
    sed 's/\${COMMIT_SHA}'"/$COMMIT_SHA/g" | \
    kubectl apply -f -
    // OR
    // Edit your images name in ./k8s/deployments/deployment.yaml and run the following command
    kubectl apply -f k8s/deployments/deployment.yaml 
    
    kubectl apply -f k8s/services/service.yaml 
    ```

2. Access k8s services via
    ```
    // minikube service SERVICE-NAME
    minikube service service-my-laravel-app
    ```