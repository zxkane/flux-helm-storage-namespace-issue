# flux-helm-storage-namespace-issue
An example repo for reproducing the FluxCD v2 specifying different release namespace and storage namespace

Add below manifest to your gitops repo to deploy Helm chart with using separated namespace for Helm storage.

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: helm-controller-issue
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/zxkane/flux-separated-helm-storage-namespace.git
  ref:
    branch: main  
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: helm-controller-issue-infra
  namespace: flux-system
spec:
  interval: 10m0s
  prune: true
  sourceRef:
    kind: GitRepository
    name: helm-controller-issue
    namespace: flux-system
  path: ./infra
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: helm-controller-issue-app
  namespace: app
spec:
  serviceAccountName: app-reconciler
  interval: 10m0s
  prune: true
  sourceRef:
    kind: GitRepository
    name: helm-controller-issue
    namespace: flux-system
  path: ./app
  dependsOn:
    - name: helm-controller-issue-infra
      namespace: flux-system
  patches:
    - patch: |-
        - op: replace
          path: /spec/serviceAccountName
          value: app-reconciler
        - op: replace
          path: /spec/storageNamespace
          value: helm-storage
      target:
        group: helm.toolkit.fluxcd.io
        version: v2beta1
        kind: HelmRelease
    - patch: |-
        - op: replace
          path: /spec/serviceAccountName
          value: app-reconciler
      target:
        group: kustomize.toolkit.fluxcd.io
        version: v1beta2
        kind: Kustomization
```
