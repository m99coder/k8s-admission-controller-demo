## Self-signed TLS certificate

_Note: execute from root directory_

```shell
# create self-signed TLS certificate for the webhook
docker run -it --rm -v ${PWD}:/work -w /work debian bash

# install Cloudflare’s PKI and TLS toolkit
apt-get update && apt-get install -y curl &&
  curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl_1.5.0_linux_amd64 -o /usr/local/bin/cfssl && \
  curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssljson_1.5.0_linux_amd64 -o /usr/local/bin/cfssljson && \
  chmod +x /usr/local/bin/cfssl && \
  chmod +x /usr/local/bin/cfssljson

# generate CA in /tmp
cfssl gencert -initca ./tls/ca-csr.json | cfssljson -bare /tmp/ca

# generate self-signed certificate in /tmp
cfssl gencert \
  -ca=/tmp/ca.pem \
  -ca-key=/tmp/ca-key.pem \
  -config=./tls/ca-config.json \
  -hostname="namespace-notifier-webhook,namespace-notifier-webhook.default.svc.cluster.local,namespace-notifier-webhook.default.svc,localhost,127.0.0.1" \
  -profile=default \
  ./tls/ca-csr.json | cfssljson -bare /tmp/namespace-notifier-webhook

# generate a secret
cat <<EOF > ./tls/namespace-notifier-webhook-tls.yaml
apiVersion: v1
kind: Secret
metadata:
  name: namespace-notifier-webhook-tls
type: Opaque
data:
  tls.crt: $(cat /tmp/namespace-notifier-webhook.pem | base64 | tr -d '\n')
  tls.key: $(cat /tmp/namespace-notifier-webhook-key.pem | base64 | tr -d '\n')
EOF

# generate CA bundle and inject it into the template
CA_PEM_BASE64="$(openssl base64 -A <"/tmp/ca.pem")"
sed -e 's@${CA_PEM_B64}@'"$CA_PEM_BASE64"'@g' <"namespace-notifier-webhook.yaml.template" \
  > namespace-notifier-webhook.yaml
```
