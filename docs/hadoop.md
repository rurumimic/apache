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

kubectl delete -f configmap.yaml -n apache
```
</details>

<details>
    <summary>Check Resources</summary>

```bash
kubectl get cm -n apache
kubectl describe cm hadoop-conf -n apache

kubectl get sts -n apache
kubectl get svc -n apache
```

</details>

<details>
    <summary>With PersistentVolume</summary>

After re-running Hadoop, NameNode may enter Safe Mode,when using a local k8s cluster such as docker desktop. 
You can use the PVs below, but I don't set them up because they're too cumbersome. 
If you turn off and turn on docker desktop, the hostPath becomes clear.

#### Start a Hadoop Cluster

```bash
kubectl apply -f namenode-storageclass.yaml
kubectl apply -f datanode-storageclass.yaml

kubectl apply -f namenode-pv.yaml
kubectl apply -f datanode-pv.yaml
```

#### Clean up Hadoop

```bash
# if use `PersistentVolume` and `volumeClaimTemplates`:
kubectl delete -n apache pvc datanode-pvc-datanode-2
kubectl delete -n apache pvc datanode-pvc-datanode-1
kubectl delete -n apache pvc datanode-pvc-datanode-0
kubectl delete -n apache pvc namenode-pvc-namenode-0

# if `persistentVolumeReclaimPolicy: Retain`:
kubectl delete -f namenode-pv.yaml
kubectl delete -f datanode-pv.yaml

kubectl delete -f namenode-storageclass.yaml
kubectl delete -f datanode-storageclass.yaml
```

#### Check Resources

```bash
kubectl get pv -n apache
kubectl get pvc -n apache
```

</details>


#### Trouble Shoot: When stuck in terminating state

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
2024-02-09 15:46:47 INFO  DefaultNoHARMFailoverProxyProvider:64 - Connecting to ResourceManager at resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local/10.1.0.24:8032
2024-02-09 15:46:47 INFO  JobResourceUploader:907 - Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1707493495914_0001
2024-02-09 15:46:47 INFO  FileInputFormat:300 - Total input files to process : 10
2024-02-09 15:46:47 INFO  JobSubmitter:202 - number of splits:10
2024-02-09 15:46:47 INFO  JobSubmitter:298 - Submitting tokens for job: job_1707493495914_0001
2024-02-09 15:46:47 INFO  JobSubmitter:299 - Executing with tokens: []
2024-02-09 15:46:48 INFO  Configuration:2854 - resource-types.xml not found
2024-02-09 15:46:48 INFO  ResourceUtils:476 - Unable to find 'resource-types.xml'.
2024-02-09 15:46:48 INFO  YarnClientImpl:338 - Submitted application application_1707493495914_0001
2024-02-09 15:46:48 INFO  Job:1682 - The url to track the job: http://resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local:8088/proxy/application_1707493495914_0001/
2024-02-09 15:46:48 INFO  Job:1727 - Running job: job_1707493495914_0001
2024-02-09 15:46:54 INFO  Job:1748 - Job job_1707493495914_0001 running in uber mode : false
2024-02-09 15:46:54 INFO  Job:1755 -  map 0% reduce 0%
2024-02-09 15:46:59 INFO  Job:1755 -  map 10% reduce 0%
2024-02-09 15:47:00 INFO  Job:1755 -  map 20% reduce 0%
2024-02-09 15:47:01 INFO  Job:1755 -  map 30% reduce 0%
2024-02-09 15:47:02 INFO  Job:1755 -  map 40% reduce 0%
2024-02-09 15:47:03 INFO  Job:1755 -  map 60% reduce 0%
2024-02-09 15:47:04 INFO  Job:1755 -  map 70% reduce 0%
2024-02-09 15:47:05 INFO  Job:1755 -  map 80% reduce 0%
2024-02-09 15:47:06 INFO  Job:1755 -  map 90% reduce 0%
2024-02-09 15:47:07 INFO  Job:1755 -  map 100% reduce 100%
2024-02-09 15:47:08 INFO  Job:1766 - Job job_1707493495914_0001 completed successfully
2024-02-09 15:47:08 INFO  Job:1773 - Counters: 54
        File System Counters
                FILE: Number of bytes read=78
                FILE: Number of bytes written=3051368
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
                Total time spent by all maps in occupied slots (ms)=17356
                Total time spent by all reduces in occupied slots (ms)=4541
                Total time spent by all map tasks (ms)=17356
                Total time spent by all reduce tasks (ms)=4541
                Total vcore-milliseconds taken by all map tasks=17356
                Total vcore-milliseconds taken by all reduce tasks=4541
                Total megabyte-milliseconds taken by all map tasks=17772544
                Total megabyte-milliseconds taken by all reduce tasks=4649984
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
                GC time elapsed (ms)=439
                CPU time spent (ms)=3450
                Physical memory (bytes) snapshot=3851857920
                Virtual memory (bytes) snapshot=29558509568
                Total committed heap usage (bytes)=4503633920
                Peak Map Physical memory (bytes)=405229568
                Peak Map Virtual memory (bytes)=2690199552
                Peak Reduce Physical memory (bytes)=299638784
                Peak Reduce Virtual memory (bytes)=2693234688
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
2024-02-09 15:47:08 INFO  DefaultNoHARMFailoverProxyProvider:64 - Connecting to ResourceManager at resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local/10.1.0.24:8032
2024-02-09 15:47:08 INFO  JobResourceUploader:907 - Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1707493495914_0002
2024-02-09 15:47:09 INFO  FileInputFormat:300 - Total input files to process : 1
2024-02-09 15:47:09 INFO  JobSubmitter:202 - number of splits:1
2024-02-09 15:47:09 INFO  JobSubmitter:298 - Submitting tokens for job: job_1707493495914_0002
2024-02-09 15:47:09 INFO  JobSubmitter:299 - Executing with tokens: []
2024-02-09 15:47:09 INFO  YarnClientImpl:338 - Submitted application application_1707493495914_0002
2024-02-09 15:47:09 INFO  Job:1682 - The url to track the job: http://resourcemanager-0.resourcemanager-svc.apache.svc.cluster.local:8088/proxy/application_1707493495914_0002/
2024-02-09 15:47:09 INFO  Job:1727 - Running job: job_1707493495914_0002
2024-02-09 15:47:13 INFO  Job:1748 - Job job_1707493495914_0002 running in uber mode : false
2024-02-09 15:47:13 INFO  Job:1755 -  map 0% reduce 0%
2024-02-09 15:47:17 INFO  Job:1755 -  map 100% reduce 0%
2024-02-09 15:47:21 INFO  Job:1755 -  map 100% reduce 100%
2024-02-09 15:47:21 INFO  Job:1766 - Job job_1707493495914_0002 completed successfully
2024-02-09 15:47:21 INFO  Job:1773 - Counters: 54
        File System Counters
                FILE: Number of bytes read=78
                FILE: Number of bytes written=553717
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=342
                HDFS: Number of bytes written=48
                HDFS: Number of read operations=9
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Launched map tasks=1
                Launched reduce tasks=1
                Rack-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=1574
                Total time spent by all reduces in occupied slots (ms)=1544
                Total time spent by all map tasks (ms)=1574
                Total time spent by all reduce tasks (ms)=1544
                Total vcore-milliseconds taken by all map tasks=1574
                Total vcore-milliseconds taken by all reduce tasks=1544
                Total megabyte-milliseconds taken by all map tasks=1611776
                Total megabyte-milliseconds taken by all reduce tasks=1581056
        Map-Reduce Framework
                Map input records=3
                Map output records=3
                Map output bytes=66
                Map output materialized bytes=78
                Input split bytes=166
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
                GC time elapsed (ms)=59
                CPU time spent (ms)=730
                Physical memory (bytes) snapshot=703115264
                Virtual memory (bytes) snapshot=5377990656
                Total committed heap usage (bytes)=751828992
                Peak Map Physical memory (bytes)=406654976
                Peak Map Virtual memory (bytes)=2685874176
                Peak Reduce Physical memory (bytes)=296460288
                Peak Reduce Virtual memory (bytes)=2692116480
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

![Datanode Replicas](/images/datanode-replicas.png)

### DataNode

![Datanode](/images/datanode.png)

<details>
    <summary>describe svc datanode-svc</summary>

```bash
kubectl describe svc -n apache datanode-svc

Name:              datanode-svc
Namespace:         apache
Labels:            <none>
Annotations:       <none>
Selector:          app=datanode
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              http  9864/TCP
TargetPort:        9864/TCP
Endpoints:         10.1.1.37:9864,10.1.1.38:9864,10.1.1.39:9864
Port:              transfer  9866/TCP
TargetPort:        9866/TCP
Endpoints:         10.1.1.37:9866,10.1.1.38:9866,10.1.1.39:9866
Port:              ipc  9867/TCP
TargetPort:        9867/TCP
Endpoints:         10.1.1.37:9867,10.1.1.38:9867,10.1.1.39:9867
Session Affinity:  None
Events:            <none>
```

</details>

### ResourceManager

![ResourceManager](/images/resourcemanager.png)

![Nodemanager replicas](/images/nodemanager-replicas.png)

### NodeManager

![NodeManager](/images/nodemanager.png)

<details>
    <summary>describe svc nodemanager-svc</summary>

```bash
kubectl describe svc -n apache nodemanager-svc

Name:              nodemanager-svc
Namespace:         apache
Labels:            <none>
Annotations:       <none>
Selector:          app=nodemanager
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              localizer  8040/TCP
TargetPort:        8040/TCP
Endpoints:         10.1.0.25:8040,10.1.0.28:8040,10.1.0.30:8040
Port:              http  8042/TCP
TargetPort:        8042/TCP
Endpoints:         10.1.0.25:8042,10.1.0.28:8042,10.1.0.30:8042
Port:              collector  8048/TCP
TargetPort:        8048/TCP
Endpoints:         10.1.0.25:8048,10.1.0.28:8048,10.1.0.30:8048
Port:              amrmproxy  8049/TCP
TargetPort:        8049/TCP
Endpoints:         10.1.0.25:8049,10.1.0.28:8049,10.1.0.30:8049
Port:              nodemanager  45454/TCP
TargetPort:        45454/TCP
Endpoints:         10.1.0.25:45454,10.1.0.28:45454,10.1.0.30:45454
Session Affinity:  None
Events:            <none>
```

</details>


### JobHistory

![JobHistory](/images/jobhistory.png)

