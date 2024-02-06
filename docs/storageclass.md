# Storage Class

## Kubernetes

### Temporary Local Sandbox

```bash
mkdir -p /tmp/apache
mkdir -m 777 /tmp/apache/namenode
mkdir -m 777 /tmp/apache/datanode
```

### Local Storage Class

- [storageclass.yaml](storageclass.yaml)

```bash
kubectl apply -f storageclass.yaml -n apache
```

```bash
kubectl delete -f storageclass.yaml -n apache
```

