# bitnami-sealed-secrets
kubernetes secret management tool -- bitnami 

[a list of kubernete secret management tool](https://argoproj.github.io/argo-cd/operator-manual/secret-management/)

[bitnami](#Bitnami Sealed Secrets)

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
### offline 使用 Kubeseal (public key / certificate 来 seal secrets)

1. 首先你的有 access 到 Kubernetes API server
2. 使用 `kubeseal` 来获取 certificate from controller （runtime 运行时）， fetch cert 命令 `kubeseal --fetch-cert > mycert.pem`
3. 在 offline 使用 `kubeseal --cert mycert.pem`
[sealed secret](https://github.com/bitnami-labs/sealed-secrets#public-key--certificate)

### 打印 public cert of kubeseal 

保存到 git repo 中， 做为 public key 来加密 
```
kubeseal --fetch-cert
```

### private key rotation

1. 下载 kubeseal controller `wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.15.0/controller.yaml -O controller.yaml`

2. 更新 kubeseal controller yaml 里面的配置，`--key-renew-period=<value>` flag for the command in the pod template of the sealed secret controller.
   ```
   containers:
     args:
       -- "--key-renew-period"
       -- "you can paste your current date time, copy/paste it here"
   ```

3. 再重新部署 kubeseal的 `kubectl apply -f controller.yaml` 后， 更新了（添加了） 新的private key
4. 查看 kube-system 的secret  `kubectl get secrets -n kube-system`, 可以看到 有多了一个 sealed-secret 在 kube-system 里面


### 打印 kubeseal log 命令

```
kubeseal -v 10 ....
```

### 重新给 sealed-secret 来加密的命令

假设 kubeseal controller 更新了 private key (在 controller side)

如何来 重新 更新 现在已经有的 sealed secret -- Re-encryption (advanced) 

```
kubeseal --re-encrypt <my_sealed_secret.json >tmp.json \
  && mv tmp.json my_sealed_secret.json

```


## 多个 集群 cluster 中使用 sealed secret 

copy private key from 1st culster 到 2nd cluster

