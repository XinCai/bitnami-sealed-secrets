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



