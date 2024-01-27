# Namespace

- [namespace.yaml](/namespace.yaml)

```bash
kubectl apply -f namespace.yaml
kubectl get ns apache
kubectl describe ns apache
kubectl delete -f namespace.yaml
```

## Create a namespace

```bash
kubectl apply -f namespace.yaml
```

```bash
namespace/apache created
```

## About namespaces

```bash
kubectl get ns
kubectl get ns apache
kubectl get namespaces
```

### Describe a namespace

```bash
kubectl describe ns apache
```

```bash
Name:         apache
Labels:       kubernetes.io/metadata.name=apache
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

## Delete a namespace

```bash
kubectl delete -f namespace.yaml
kubectl delete ns apache
```

