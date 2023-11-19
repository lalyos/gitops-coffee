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
k apply -k https://github.com/lalyos/gitops-coffee.git/?ref=stage2
```

### Stage-3: manual git-source + kustomization(flux)

[sample git repo](https://github.com/lalyos/gitops-coffee/tree/stage2)

```
flux create source git gitops-coffee-stage2 \
  --url https://github.com/lalyos/gitops-coffee.git \
  --branch stage2

flux create kustomization coffee-stage2 \
  --source gitrepository/gitops-coffee-stage2
```

List all related flux Custom Resources:
```
flux tree kustomization coffee-stage2
```

### Stage-4 combined git-source + kustomization from git repo


combine the 2 previous commands `--export` output into 1 single `gitops-stage3.yaml`
and add it to the repo.
[sample git repo](https://github.com/lalyos/gitops-coffee/tree/stage3)

```
k apply -f https://raw.githubusercontent.com/lalyos/gitops-coffee/stage3/gitops-stage3.yaml
```

### Stage-5: every yaml in git repo:

- app
  - HelmRepo
  - HelmRelease
- infra
  - GitSource (for itself)
  - Kustomization (flux, pointing to app dir)