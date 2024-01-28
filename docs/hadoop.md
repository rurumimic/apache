# Hadoop

- [hadoop/](/hadoop)
  - [configmap.yaml](/hadoop/configmap.yaml)
  - [namenode.yaml](/hadoop/namenode.yaml)
  - [datanode.yaml](/hadoop/datanode.yaml)

```bash
kubectl apply -f configmap.yaml -n apache
kubectl apply -f namenode.yaml -n apache
kubectl apply -f datanode.yaml -n apache
```

```bash
kubectl delete -f configmap.yaml -n apache
kubectl delete -f namenode.yaml -n apache
kubectl delete -f datanode.yaml -n apache
```

