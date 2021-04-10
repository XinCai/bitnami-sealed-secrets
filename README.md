# bitnami-sealed-secrets
kubernetes secret management tool -- bitnami 

[a list of kubernete secret management tool](https://argoproj.github.io/argo-cd/operator-manual/secret-management/)


### Bitnami Sealed Secrets
[Bitnami offcial website](https://github.com/bitnami-labs/sealed-secrets#sealed-secrets-for-kubernetes)

#### Installation 

```
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
```

#### 客户端工具 Installing the kubeseal client

Client side
Install client-side tool into /usr/local/bin/:

For Linux x86_64 systems, the client-tool may be installed into /usr/local/bin with the following command:

```
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.15.0/kubeseal-linux-amd64 -O kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

Macos: (might lag a few hours behind a new release, this icon will reflect that latest release)
```
brew install kubeseal
```

### 在集群方向安装 (Installing the custom controller and CRD for SealedSecret)

Install SealedSecret CRD, server-side controller into kube-system namespace.

```
$ kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.15.0/controller.yaml

```

#### 常用命令
```
# Create a json/yaml-encoded Secret somehow:
# (note use of `--dry-run` - this is just a local file!)
$ echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json >mysecret.json

# This is the important bit:
# (note default format is json!)
$ kubeseal <mysecret.json >mysealedsecret.json

# mysealedsecret.json is safe to upload to github, post to twitter,
# etc.  Eventually:
$ kubectl create -f mysealedsecret.json

# Profit!
$ kubectl get secret mysecret
```
### 打印 public cert of kubeseal 

```
kubeseal --fetch-cert
```
