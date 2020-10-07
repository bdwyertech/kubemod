# KubeMod

KubeMod is a universal Kubernetes resource mutator.

It allows you to deploy to your Kubernetes cluster a set of declarative rules which will  perform targeted modifications
to any Kubernetes resource at the time the resource is being created or updated.

## Installation

KubeMod is an implementation of a [Kubernetes Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/).

To install KubeMod, run:

```bash
kubectl apply -f https://github.com/kubemod/kubemod/blob/v0.4.0/bundle.yaml
```

To uninstall KubeMod, run:

```bash
kubectl delete -f https://github.com/kubemod/kubemod/blob/v0.4.0/bundle.yaml
```

## Getting started

Once KubeMod is deployed, you can create `ModRules` which monitor for specific resources and perform modifications on them.

For example, here's a ModRule which enforces a `securityContext` and adds annotation `my-annotation` to any `Deployment`
resource whose `app` label equals `nginx` and includes a container of any subversion of nginx `1.14`.

```yaml
apiVersion: api.kubemod.io/v1beta1
kind: ModRule
metadata:
  name: my-modrule
spec:
  type: Patch

  matches:
    # Match deployments ...
    - query: '$.kind'
      value: 'Deployment'
    #... with label app=nginx ...
    - query: '$.metadata.labels.app'
      regex: 'nginx'
    #... and containers whose image matches nginx:1.14.*
    - query: '$.spec.template.spec.containers[*].image'
      regex: 'nginx:1\.14\..*'
    
  patch:
    # Add custom annotation.
    - op: add
      path: /metadata/annotations/my-annotation
      value: whatever

    # Enforce non-root securityContext and make nginx run as user 101.
    - op: add
      path: /spec/template/spec/securityContext
      value: |-
        fsGroup: 101
        runAsGroup: 101
        runAsUser: 101
        runAsNonRoot: true
```
 
 Save the above to file `my-modrule.yaml` and run:
 ```bash
 kubectl apply -f my-modrule.yaml
```

This will create ModRule `my-modrule` in the current default namespace.
 
After the ModRule is created, the creation of any nginx Kubernetes Deployment resource in the same namespace will be intercepted by KubeMod and if
the Deployment resource matches all the queries in the ModRule's `matches` section the resource will be patched with the `patch` operations **before**
it is actually deployed to Kubernetes.

See more examples of ModRules [here](https://github.com/kubemod/kubemod/tree/master/core/testdata/modrules).

## Contributing

Contributions are greatly appreciated! Leave [an issue](https://github.com/kubemod/kubemod/issues)
or [create a PR](https://github.com/kubemod/kubemod/compare).

### Development Prerequisites

* kubebuilder (2.3.1) (https://book.kubebuilder.io/quick-start.html)
* kustomize (3.8.1) (https://kubernetes-sigs.github.io/kustomize/installation/binaries/)
* cfssl (https://gist.github.com/tirumaraiselvan/b7eb1831d25dd9d59a785c11bd46c84b)
* telepresence (https://www.telepresence.io/)
* wire (https://github.com/google/wire)

### Dev cycle

Build the image once:
```bash
make docker-build
```
then deploy the whole shebang and start telepresence which will swap out the kubemod controller with your local host environment:
```
make deploy
dev-env.sh
```
At this point you can develop/debug kubemod locally.