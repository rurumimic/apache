# Hadoop

## Kubernetes

- [hadoop/](/hadoop)
  - [configmap.yaml](/hadoop/configmap.yaml)
  - [namenode.yaml](/hadoop/namenode.yaml)
  - [datanode.yaml](/hadoop/datanode.yaml)
  - [namenode-pv.yaml](/hadoop/namenode-pv.yaml)
  - [datanode-pv.yaml](/hadoop/datanode-pv.yaml)

### Persistent Volume on Host

```bash
mkdir -p /tmp/apache
mkdir -m 777 /tmp/apache/namenode
mkdir -m 777 /tmp/apache/datanode
```

### Start Hadoop

```bash
kubectl apply -f configmap.yaml -n apache

kubectl apply -f namenode-storageclass.yaml -n apache
kubectl apply -f datanode-storageclass.yaml -n apache

kubectl apply -f namenode-pv.yaml -n apache
kubectl apply -f datanode-pv.yaml -n apache

kubectl apply -f namenode.yaml -n apache
kubectl apply -f datanode.yaml -n apache
kubectl apply -f resourcemanager.yaml -n apache
kubectl apply -f nodemanager.yaml -n apache
kubectl apply -f jobhistory.yaml -n apache
```

### Check Resources

```bash
kubectl get cm -n apache
kubectl describe cm hadoop-conf -n apache

kubectl get pv -n apache
kubectl get pvc -n apache
kubectl describe pv namenode-pv -n apache
kubectl describe pv datanode-pv -n apache

kubectl get sts -n apache
kubectl get svc -n apache
```

### Stop Hadoop

```bash
kubectl delete -f jobhistory.yaml -n apache
kubectl delete -f nodemanager.yaml -n apache
kubectl delete -f resourcemanager.yaml -n apache
kubectl delete -f datanode.yaml -n apache
kubectl delete -f namenode.yaml -n apache

kubectl delete -f namenode-pv.yaml -n apache
kubectl delete -f datanode-pv.yaml -n apache

kubectl delete -f namenode-storageclass.yaml -n apache
kubectl delete -f datanode-storageclass.yaml -n apache

kubectl delete -f configmap.yaml -n apache
```

#### When stuck in terminating state

```bash
kubectl get pv -n apache

NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                            STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
namenode-pv   128Mi      RWO            Retain           Terminating   apache/namenode-pvc-namenode-0   local-storage   <unset>                          13m
```

##### option 1

```bash
kubectl patch -n apache pv namenode-pv -p '{"metadata":{"finalizers":null}}'

persistentvolume/namenode-pv patched
```

##### option 2

```bash
kubectl edit -n apache pv namenode-pv
```

Remove the finalizers section and save the file.

```yaml
# ...
  finalizers:
  - kubernetes.io/pv-protection
# ...
```

---

## Docker Compose

- [hadoop/docker](/hadoop/docker)
  - [compose.yaml](/hadoop/docker/compose.yaml)

### Start Hadoop

```bash
cd hadoop/docker
docker compose up -d
```

---

## Examples

### Test a MapReduce job

#### Check Resources

```bash
kubectl get sts -n apache

NAME              READY
datanode          1/1
jobhistory        1/1
namenode          1/1
nodemanager       1/1
resourcemanager   1/1
```

#### Attach to Namenode

```bash
kubectl exec --stdin --tty sts/namenode -n apache -- /bin/bash
docker compose exec namenode bash
```

#### Make Working Directory

```bash
whoami # hadoop
# hdfs dfs -mkdir -p /user/$USER
hdfs dfs -mkdir -p /user/hadoop
```

#### Load test files

```bash
hdfs dfs -mkdir input
hdfs dfs -put $HADOOP_HOME/etc/hadoop/*.xml input
```

View the input files:

```bash
hdfs dfs -ls -R /
```

<details>
    <summary>List HDFS</summary>

```bash
drwxrwx---   - hadoop supergroup          0 2024-02-06 14:45 /tmp
drwxrwx---   - hadoop supergroup          0 2024-02-06 14:45 /tmp/hadoop-yarn
drwxrwx---   - hadoop supergroup          0 2024-02-06 14:45 /tmp/hadoop-yarn/staging
drwxrwx---   - hadoop supergroup          0 2024-02-06 14:45 /tmp/hadoop-yarn/staging/history
drwxrwx---   - hadoop supergroup          0 2024-02-06 14:45 /tmp/hadoop-yarn/staging/history/done
drwxrwxrwt   - hadoop supergroup          0 2024-02-06 14:45 /tmp/hadoop-yarn/staging/history/done_intermediate
drwxr-xr-x   - hadoop supergroup          0 2024-02-06 14:56 /user
drwxr-xr-x   - hadoop supergroup          0 2024-02-06 14:56 /user/hadoop
drwxr-xr-x   - hadoop supergroup          0 2024-02-06 14:56 /user/hadoop/input
-rw-r--r--   1 hadoop supergroup       1402 2024-02-06 14:56 /user/hadoop/input/capacity-scheduler.xml
-rw-r--r--   1 hadoop supergroup        189 2024-02-06 14:56 /user/hadoop/input/core-site.xml
-rw-r--r--   1 hadoop supergroup      11765 2024-02-06 14:56 /user/hadoop/input/hadoop-policy.xml
-rw-r--r--   1 hadoop supergroup        683 2024-02-06 14:56 /user/hadoop/input/hdfs-rbf-site.xml
-rw-r--r--   1 hadoop supergroup        185 2024-02-06 14:56 /user/hadoop/input/hdfs-site.xml
-rw-r--r--   1 hadoop supergroup        620 2024-02-06 14:56 /user/hadoop/input/httpfs-site.xml
-rw-r--r--   1 hadoop supergroup       3518 2024-02-06 14:56 /user/hadoop/input/kms-acls.xml
-rw-r--r--   1 hadoop supergroup        682 2024-02-06 14:56 /user/hadoop/input/kms-site.xml
-rw-r--r--   1 hadoop supergroup        589 2024-02-06 14:56 /user/hadoop/input/mapred-site.xml
-rw-r--r--   1 hadoop supergroup        724 2024-02-06 14:56 /user/hadoop/input/yarn-site.xml
```

</details>

#### Run a MapReduce job

Check your hadoop version:

```bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.y.z.jar \
grep input output 'dfs[a-z.]+'
```

#### View output files

```bash
hdfs dfs -cat output/*
```

```bash
1       dfsadmin
1       dfs.replication
1       dfs.namenode.rpc
```

---

## Web UI

- Namenode
  - kube: [http://localhost:30870](http://localhost:30870)
  - docker: [http://localhost:9870](http://localhost:9870)
- Datanode
  - kube: [http://localhost:30864](http://localhost:30864)
  - docker: [http://localhost:9864](http://localhost:9864)
- ResourceManager
  - kube: [http://localhost:30088](http://localhost:30088)
  - docker: [http://localhost:8088](http://localhost:8088)
- NodeManager
  - kube: [http://localhost:30042](http://localhost:30042)
  - docker: [http://localhost:8042](http://localhost:8042)
- JobHistory
  - kube: [http://localhost:30888](http://localhost:30888)
  - docker: [http://localhost:19888](http://localhost:19888)

### Namenode

![Namenode](/images/namenode.png)

### DataNode

![Datanode](/images/datanode.png)

### ResourceManager

![ResourceManager](/images/resourcemanager.png)

### NodeManager

![NodeManager](/images/nodemanager.png)

### JobHistory

![JobHistory](/images/jobhistory.png)

