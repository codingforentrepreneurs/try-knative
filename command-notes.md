

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


### Verify Environment Variable

```bash

export KNATIVE_SERIVCE_NAME="cfe-nginx"
export K8S_DEPLOYMENT_NAME="example-deployment"

export KNATIVE_SERVICE_URL=$(kubectl get ksvc $KNATIVE_SERIVCE_NAME -o jsonpath='{.status.address.url}')

echo $KNATIVE_SERVICE_URL

export LAST_REVISION_NAME=$(kubectl get ksvc $KNATIVE_SERIVCE_NAME -o jsonpath='{.status.latestReadyRevisionName}')

echo $LAST_REVISION_NAME

kubectl exec -n default -it deployments/$K8S_DEPLOYMENT_NAME -- bash -c "curl $KNATIVE_SERVICE_URL"

kubectl exec -n default -it "deployments/$LAST_REVISION_NAME-deployment" -- bash -c 'echo $DJANGO_SECRET_KEY'

kubectl exec -n default -it "deployments/$LAST_REVISION_NAME-deployment" -- bash -c "echo \$DJANGO_SECRET_KEY"
```