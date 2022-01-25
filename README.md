# Kubernetes Admission Controller

> Based on [How to build a Kubernetes Webhook | Admission controllers](https://www.youtube.com/watch?v=1mNYSn2KMZk)

## Local Kuberbetes cluster

```shell
# create kubernetes cluster in docker
kind create cluster --name webhook --image kindest/node:v1.20.2
kubectl cluster-info --context kind-webhook
```
