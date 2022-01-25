# Kubernetes Admission Controller

> Based on [How to build a Kubernetes Webhook | Admission controllers](https://www.youtube.com/watch?v=1mNYSn2KMZk)

## Local Kuberbetes cluster

```shell
# create kubernetes cluster in docker
kind create cluster --name admission-controller --image kindest/node:v1.20.2
kubectl cluster-info --context kind-admission-controller

# delete kubernetes cluster
kind delete cluster --name admission-controller
```

## Build and run docker image locally

```shell
cd src
docker build . -t namespace-notifier
docker run -it -e USE_KUBECONFIG=true --rm --net host -v ${HOME}/.kube/:/root/.kube/ -v ${PWD}:/app namespace-notifier sh
```

## Build and push docker image

```shell
cd src
docker build . -t m99coder/namespace-notifier-webhook:v1
docker push m99coder/namespace-notifier-webhook:v1
```

## Deployment

```shell
# apply secret, rbac, service and deployment
kubectl -n default apply -f ./tls/namespace-notifier-webhook-tls.yaml
kubectl -n default apply -f rbac.yaml
kubectl -n default apply -f deployment.yaml

# check running pods and only then apply the webhook
kubectl -n default get pods
kubectl -n default apply -f namespace-notifier-webhook.yaml
```

## Demo

```shell
# apply namespace
kubectl create -f demo-namespace.yaml

# check logs
WEBHOOK_POD_NAME=`kubectl -n default get pods -l app=namespace-notifier-webhook -o json | jq -r '.items[0].metadata.name'`
kubectl logs $WEBHOOK_POD_NAME
```

## Resources

- [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Sample Admission Review Request](./demo-namespace-request.json)
