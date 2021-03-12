# bitnami-sealed-secrets
kubernetes secret management tool -- bitnami 

[a list of kubernete secret management tool](https://argoproj.github.io/argo-cd/operator-manual/secret-management/)


### Bitnami Sealed Secrets
[Bitnami offcial website](https://github.com/bitnami-labs/sealed-secrets#sealed-secrets-for-kubernetes)

#### Installation 
```
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
```


#### Usage
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
