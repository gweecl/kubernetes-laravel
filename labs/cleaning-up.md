# Cleaning Up

Deleting GKE cluster from the GCP console will also remove the following:
* Compute instances for the cluster
* Load balancer for the Ingress
* Network IPs or External IP addresses

Image pushed to the container registry will remain intact.


## Kubernetes

### `kubectl`

View contexts.
```
kubectl config get-contexts
```
Clear contexts in `kubectl`.
```
kubectl config unset contexts.CONTEXT_NAME
```

### Helm
```
helm del --purge CHART_NAME
```
