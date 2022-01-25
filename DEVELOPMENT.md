## Webhook

```shell
# build docker image
cd src
docker build . -t namespace-notifier

# run in `host` network and mount k8s config
docker run -it --rm --net host -v ${HOME}/.kube/:/root/.kube/ -v ${PWD}:/app namespace-notifier sh

# install kubectl
apk add --no-cache curl
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl

# check connectivity
kubectl get nodes

# init module
# to update run `go mod tidy`
go mod init namespace-notifier

# build binary
export CGO_ENABLED=0
go build -o namespace-notifier

# run binary
export USE_KUBECONFIG=true
./namespace-notifier
```
