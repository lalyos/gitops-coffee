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