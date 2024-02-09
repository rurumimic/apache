# Hadoop

- source: [/hadoop](/hadoop)
- hadoop [docs](https://hadoop.apache.org/docs/current/)
  - [core-default.xml](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml)
  - [hdfs-default.xml](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)
  - [mapred-default.xml](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)
  - [yarn-default.xml](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-common/yarn-default.xml)

## Kubernetes

### Commands

<details>
    <summary>Start a Hadoop Cluster</summary>

```bash
kubectl apply -f configmap.yaml -n apache

kubectl apply -f namenode-storageclass.yaml
kubectl apply -f datanode-storageclass.yaml

kubectl apply -f namenode-pv.yaml
kubectl apply -f datanode-pv.yaml

kubectl apply -f namenode-service.yaml -n apache
kubectl apply -f datanode-service.yaml -n apache
kubectl apply -f resourcemanager-service.yaml -n apache
kubectl apply -f nodemanager-service.yaml -n apache
kubectl apply -f jobhistory-service.yaml -n apache

kubectl apply -f namenode-statefulset.yaml -n apache
kubectl apply -f datanode-statefulset.yaml -n apache
kubectl apply -f resourcemanager-statefulset.yaml -n apache
kubectl apply -f nodemanager-statefulset.yaml -n apache
kubectl apply -f jobhistory-statefulset.yaml -n apache
```

</details>

<details>
    <summary>Clean up Hadoop</summary>

```bash
kubectl delete -f jobhistory-statefulset.yaml -n apache
kubectl delete -f nodemanager-statefulset.yaml -n apache
kubectl delete -f resourcemanager-statefulset.yaml -n apache
kubectl delete -f datanode-statefulset.yaml -n apache
kubectl delete -f namenode-statefulset.yaml -n apache

kubectl delete -f jobhistory-service.yaml -n apache
kubectl delete -f nodemanager-service.yaml -n apache
kubectl delete -f resourcemanager-service.yaml -n apache
kubectl delete -f datanode-service.yaml -n apache
kubectl delete -f namenode-service.yaml -n apache

kubectl delete -n apache pvc datanode-pvc-datanode-0
kubectl delete -n apache pvc namenode-pvc-namenode-0

kubectl delete -f namenode-pv.yaml
kubectl delete -f datanode-pv.yaml

kubectl delete -f namenode-storageclass.yaml
kubectl delete -f datanode-storageclass.yaml

kubectl delete -f configmap.yaml -n apache
```

</details>

<details>
    <summary>Check Resources</summary>

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

</details>

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

<details>
    <summary>MapReduce Job Logs</summary>

```log
2024-02-09 11:15:26 INFO  DefaultNoHARMFailoverProxyProvider:64 - Connecting to ResourceManager at resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local/10.1.1.26:8032
2024-02-09 11:15:26 INFO  JobResourceUploader:907 - Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1707477277198_0001
2024-02-09 11:15:27 INFO  FileInputFormat:300 - Total input files to process : 10
2024-02-09 11:15:27 INFO  JobSubmitter:202 - number of splits:10
2024-02-09 11:15:27 INFO  JobSubmitter:298 - Submitting tokens for job: job_1707477277198_0001
2024-02-09 11:15:27 INFO  JobSubmitter:299 - Executing with tokens: []
2024-02-09 11:15:27 INFO  Configuration:2854 - resource-types.xml not found
2024-02-09 11:15:27 INFO  ResourceUtils:476 - Unable to find 'resource-types.xml'.
2024-02-09 11:15:27 INFO  YarnClientImpl:338 - Submitted application application_1707477277198_0001
2024-02-09 11:15:27 INFO  Job:1682 - The url to track the job: http://resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local:8088/proxy/application_1707477277198_0001/
2024-02-09 11:15:27 INFO  Job:1727 - Running job: job_1707477277198_0001
2024-02-09 11:15:33 INFO  Job:1748 - Job job_1707477277198_0001 running in uber mode : false
2024-02-09 11:15:33 INFO  Job:1755 -  map 0% reduce 0%
2024-02-09 11:15:38 INFO  Job:1755 -  map 10% reduce 0%
2024-02-09 11:15:39 INFO  Job:1755 -  map 20% reduce 0%
2024-02-09 11:15:40 INFO  Job:1755 -  map 30% reduce 0%
2024-02-09 11:15:41 INFO  Job:1755 -  map 40% reduce 0%
2024-02-09 11:15:42 INFO  Job:1755 -  map 60% reduce 0%
2024-02-09 11:15:44 INFO  Job:1755 -  map 70% reduce 0%
2024-02-09 11:15:45 INFO  Job:1755 -  map 90% reduce 0%
2024-02-09 11:15:46 INFO  Job:1755 -  map 100% reduce 0%
2024-02-09 11:15:47 INFO  Job:1755 -  map 100% reduce 100%
2024-02-09 11:15:48 INFO  Job:1766 - Job job_1707477277198_0001 completed successfully
2024-02-09 11:15:49 INFO  Job:1773 - Counters: 54
        File System Counters
                FILE: Number of bytes read=78
                FILE: Number of bytes written=3051346
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=22141
                HDFS: Number of bytes written=176
                HDFS: Number of read operations=35
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Launched map tasks=10
                Launched reduce tasks=1
                Rack-local map tasks=10
                Total time spent by all maps in occupied slots (ms)=16013
                Total time spent by all reduces in occupied slots (ms)=5738
                Total time spent by all map tasks (ms)=16013
                Total time spent by all reduce tasks (ms)=5738
                Total vcore-milliseconds taken by all map tasks=16013
                Total vcore-milliseconds taken by all reduce tasks=5738
                Total megabyte-milliseconds taken by all map tasks=16397312
                Total megabyte-milliseconds taken by all reduce tasks=5875712
        Map-Reduce Framework
                Map input records=506
                Map output records=3
                Map output bytes=66
                Map output materialized bytes=132
                Input split bytes=1539
                Combine input records=3
                Combine output records=3
                Reduce input groups=3
                Reduce shuffle bytes=132
                Reduce input records=3
                Reduce output records=3
                Spilled Records=6
                Shuffled Maps =10
                Failed Shuffles=0
                Merged Map outputs=10
                GC time elapsed (ms)=467
                CPU time spent (ms)=3610
                Physical memory (bytes) snapshot=3573338112
                Virtual memory (bytes) snapshot=29551747072
                Total committed heap usage (bytes)=4665638912
                Peak Map Physical memory (bytes)=402853888
                Peak Map Virtual memory (bytes)=2688602112
                Peak Reduce Physical memory (bytes)=299581440
                Peak Reduce Virtual memory (bytes)=2693697536
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=20602
        File Output Format Counters
                Bytes Written=176
2024-02-09 11:15:49 INFO  DefaultNoHARMFailoverProxyProvider:64 - Connecting to ResourceManager at resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local/10.1.1.26:8032
2024-02-09 11:15:49 INFO  JobResourceUploader:907 - Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1707477277198_0002
2024-02-09 11:15:49 INFO  FileInputFormat:300 - Total input files to process : 1
2024-02-09 11:15:49 INFO  JobSubmitter:202 - number of splits:1
2024-02-09 11:15:49 INFO  JobSubmitter:298 - Submitting tokens for job: job_1707477277198_0002
2024-02-09 11:15:49 INFO  JobSubmitter:299 - Executing with tokens: []
2024-02-09 11:15:49 INFO  YarnClientImpl:338 - Submitted application application_1707477277198_0002
2024-02-09 11:15:49 INFO  Job:1682 - The url to track the job: http://resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local:8088/proxy/application_1707477277198_0002/
2024-02-09 11:15:49 INFO  Job:1727 - Running job: job_1707477277198_0002
2024-02-09 11:15:58 INFO  Job:1748 - Job job_1707477277198_0002 running in uber mode : false
2024-02-09 11:15:58 INFO  Job:1755 -  map 0% reduce 0%
2024-02-09 11:16:03 INFO  Job:1755 -  map 100% reduce 0%
2024-02-09 11:16:07 INFO  Job:1755 -  map 100% reduce 100%
2024-02-09 11:16:07 INFO  Job:1766 - Job job_1707477277198_0002 completed successfully
2024-02-09 11:16:07 INFO  Job:1773 - Counters: 54
        File System Counters
                FILE: Number of bytes read=78
                FILE: Number of bytes written=553713
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=340
                HDFS: Number of bytes written=48
                HDFS: Number of read operations=9
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Launched map tasks=1
                Launched reduce tasks=1
                Rack-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=1562
                Total time spent by all reduces in occupied slots (ms)=1493
                Total time spent by all map tasks (ms)=1562
                Total time spent by all reduce tasks (ms)=1493
                Total vcore-milliseconds taken by all map tasks=1562
                Total vcore-milliseconds taken by all reduce tasks=1493
                Total megabyte-milliseconds taken by all map tasks=1599488
                Total megabyte-milliseconds taken by all reduce tasks=1528832
        Map-Reduce Framework
                Map input records=3
                Map output records=3
                Map output bytes=66
                Map output materialized bytes=78
                Input split bytes=164
                Combine input records=0
                Combine output records=0
                Reduce input groups=1
                Reduce shuffle bytes=78
                Reduce input records=3
                Reduce output records=3
                Spilled Records=6
                Shuffled Maps =1
                Failed Shuffles=0
                Merged Map outputs=1
                GC time elapsed (ms)=53
                CPU time spent (ms)=640
                Physical memory (bytes) snapshot=706383872
                Virtual memory (bytes) snapshot=5386444800
                Total committed heap usage (bytes)=757071872
                Peak Map Physical memory (bytes)=405561344
                Peak Map Virtual memory (bytes)=2689998848
                Peak Reduce Physical memory (bytes)=300822528
                Peak Reduce Virtual memory (bytes)=2696445952
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=176
        File Output Format Counters
                Bytes Written=48
```

</details>

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

