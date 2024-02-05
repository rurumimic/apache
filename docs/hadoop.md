# Hadoop

## Kubernetes

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

---

## Docker Compose

- [hadoop/docker](/hadoop/docker)
  - [compose.yaml](/hadoop/docker/compose.yaml)

### Start Hadoop

```bash
cd hadoop/docker
docker compose up -d
```

### Test a MapReduce job


#### Attach to Namenode

```bash
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

#### Run a MapReduce job

Check your hadoop version:

```bash
hadoop jar \
$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.y.z.jar \
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

- Namenode: [http://localhost:9870](http://localhost:9870)
- Datanode: [http://localhost:9864](http://localhost:9864)
- ResourceManager: [http://localhost:8088](http://localhost:8088)
- NodeManager: [http://localhost:8042](http://localhost:8042)
- JobHistory: [http://localhost:8188](http://localhost:8188)

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

