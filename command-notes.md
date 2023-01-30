

## SSH into a Deployment
```
kubectl exec -n default -it example-deployment -- /bin/bash
```
> `Error from server (NotFound): pods "example-deployment" not found`

```
kubectl exec -n default -it deployments/example-deployment -- /bin/bash
```


## Get our Knative Service URL

```
kubectl get ksvc
```


## Curl to a Kubernetes Service

```
curl <knative service url>
```

```
curl http://cfe-nginx.default.svc.cluster.local
```