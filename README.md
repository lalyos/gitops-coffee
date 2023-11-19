Demo of gitops with [flux](https://fluxcd.io/flux/components/)

## Setup

```
k apply -k https://github.com/lalyos/gitops-coffee/infra
```

## Test Reconciliation

Try to manually change one of the helm releases:
```
k patch \
  helmreleases dinner \
  --patch='[{
      "op": "replace",
      "path": "/spec/values/color",
      "value": "blue"
    }]' --type json
```

Watch for changes
```
k get hr -w
# or
k get po -n lalyos --watch-only
```

## tl;dr

### Stage-1: manual helmrelease

[git](https://github.com/lalyos/gitops-coffee/commit/4b3927908d45cc7385ca249671aee337cfb07951#diff-d7b0a7e1ae06cfedb2dc7a10f6ecb8212f28d091361ae9b7e747e66499ad0cdf)

```
flux create source helm lalyos \
  --url https://chart.lalyo.sh/ 

flux create helmrelease coffee \
  --source HelmRepository/lalyos \
  --chart 12factor \
  --target-namespace=lalyos \
  --create-target-namespace=true \
  --values ./values.yaml
```

### Stage-2: helmrelease from git

Repeate previous commands with `--export > xxx.yaml` and put into a git repo:
  - HelmRepo
  - HelmRelease
  - kustomization(standard)

```
k apply -k http://github.com/user/gitops
```

### Stage-3: manual git-source + kustomization(flux)



```
flux create source git gitops-coffee \
  --url https://github.com/lalyos/gitops-coffee.git \
  --branch master

flux create kustomization coffee \
  --source gitrepository/gitops-coffee
```

