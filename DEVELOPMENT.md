## Webhook

```shell
# build docker image
cd src
docker build . -t webhook

# run in `host` network and mount k8s config
docker run -it --rm --net host -v ${HOME}/.kube/:/root/.kube/ -v ${PWD}:/app webhook sh

# install kubectl
apk add --no-cache curl
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl

# check connectivity
kubectl get nodes

# init module
# to update run `go mod tidy`
go mod init example-webhook

# build binary
export CGO_ENABLED=0
go build -o webhook

# run binary
export USE_KUBECONFIG=true
./webhook
```

## Deployment

```shell
# build namespaced and versioned docker image
cd src
docker build . -t m99coder/example-webhook:v1
docker push m99coder/example-webhook:v1

# apply generated secret
kubectl -n default apply -f ./tls/example-webhook-tls.yaml

# service account and deployment
kubectl -n default apply -f rbac.yaml
kubectl -n default apply -f deployment.yaml

# check running pods and only then deploy the webhook
kubectl -n default get pods
kubectl -n default apply -f webhook.yaml

# check logs
WEBHOOK_POD_NAME=`kubectl -n default get pods -l app=example-webhook -o json | jq -r '.items[0].metadata.name'`
kubectl logs $WEBHOOK_POD_NAME
```

## Demo application

```shell
# deploy demo application
# NOTE: this will result in an error because the webhook isn’t returned the expected response
kubectl -n default apply -f demo-pod.yaml

# copy the request Kubernetes sent to the webhook
kubectl cp $WEBHOOK_POD_NAME:/tmp/request ./example-request.json
```

## Build and push updates

```shell
cd src
docker build . -t m99coder/example-webhook:v1
docker push m99coder/example-webhook:v1
```

## Re-deploy and see the mutation

```shell
# delete all pods and re-deploy the demo application
kubectl delete pods --all
kubectl -n default apply -f demo-pod.yaml

# list pods with labels
kubectl get pods --show-labels

# check logs
WEBHOOK_POD_NAME=`kubectl -n default get pods -l app=example-webhook -o json | jq -r '.items[0].metadata.name'`
kubectl logs $WEBHOOK_POD_NAME
```

## Logs

```
There are 10 pods in the cluster
Type: /v1, Kind=Pod      Event: CREATE   Name: demo-pod
```
