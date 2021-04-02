
Redis Cluster requires at least 3 master nodes, so we have 6 replicas here ( 3 master / 3 slave )

```
kubectl apply -f 00-namespace.yaml
kubectl apply -f 02-deployment.yaml
```

## Make cluster
You should scale the cluster up or down manually
### Get cluster nodes

```
kubectl get pods -l name=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '
```

Now we are going to create the cluster ( assign master/slave roles, distribute slot maps) using the `redis-cli --cluster create` script.  
Since we are going create 3 masters cluster with 3 dedicated slaves, the `--cluster-replicas 1` flag is passed.

```
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 <<< node list from previous command >>>
```

Or you can merge command like this  then copy ip result

```
kubectl get pods -l name=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '

172.17.0.13:6379 172.17.0.14:6379 172.17.0.16:6379 172.17.0.18:6379 172.17.0.19:6379 172.17.0.21:6379
```


```
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 172.17.0.13:6379 172.17.0.14:6379 172.17.0.16:6379 172.17.0.18:6379 172.17.0.19:6379 172.17.0.21:6379
```


```
/data # redis-cli --cluster create --cluster-replicas 1 172.17.0.13:6379 172.17.0.14:6379 172.17.0.16:6379 172.17.0.18:6379 172.17.0.19:6379 172.17.0.21:6379
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.17.0.19:6379 to 172.17.0.13:6379
Adding replica 172.17.0.21:6379 to 172.17.0.14:6379
Adding replica 172.17.0.18:6379 to 172.17.0.16:6379
M: 34838a169519141130ecdc0b768a3fe9907ad6bd 172.17.0.13:6379
   slots:[0-5460] (5461 slots) master
M: d6607fb3106a95a1595495545e6d051bfa4a5d90 172.17.0.14:6379
   slots:[5461-10922] (5462 slots) master
M: 8fbb718ef8519bbf98708c55056d71437c4aa2e3 172.17.0.16:6379
   slots:[10923-16383] (5461 slots) master
S: 140d23b950b9444e280e91a6b5971404080430b0 172.17.0.18:6379
   replicates 8fbb718ef8519bbf98708c55056d71437c4aa2e3
S: f273bedbde988622c2b25291b00ce3b82382ecce 172.17.0.19:6379
   replicates 34838a169519141130ecdc0b768a3fe9907ad6bd
S: ac2d02b42afcb68e5a13397cf2ee04da0aa3c78f 172.17.0.21:6379
   replicates d6607fb3106a95a1595495545e6d051bfa4a5d90
Can I set the above configuration? (type 'yes' to accept): yes
```

## 3 master / 3 slave
## Redis Benchmark
### GET
```bash
$ redis-benchmark -t get -c 10000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== GET ======
  100000 requests completed in 7.13 seconds
  10000 parallel clients
  3 bytes payload
  keep alive: 1
14019.35 requests per second
```
### SET
```bash
$ redis-benchmark -t set -c 10000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== SET ======
  100000 requests completed in 7.05 seconds
  10000 parallel clients
  3 bytes payload
  keep alive: 1
14190.44 requests per second
```

### GET
```bash
$ redis-benchmark -t get -c 100000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== GET ======
  100000 requests completed in 6.80 seconds
  100000 parallel clients
  3 bytes payload
  keep alive: 1
14703.72 requests per second

```
### SET
```bash
$ redis-benchmark -t set -c 100000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== SET ======
  100000 requests completed in 7.02 seconds
  100000 parallel clients
  3 bytes payload
  keep alive: 1
14242.99 requests per second
```


## 6 master / 6 slave
### GET
```bash
$ redis-benchmark -t get -c 10000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== GET ======
  100000 requests completed in 6.91 seconds
  10000 parallel clients
  3 bytes payload
  keep alive: 1
14461.32 requests per second
```
### SET
```bash
$ redis-benchmark -t set -c 10000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== SET ======
  100000 requests completed in 7.78 seconds
  10000 parallel clients
  3 bytes payload
  keep alive: 1
12846.87 requests per second
```

### GET
```bash
redis-benchmark -t get -c 100000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== GET ======
  100000 requests completed in 8.46 seconds
  100000 parallel clients
  3 bytes payload
  keep alive: 1
11816.14 requests per second
```
### SET
```bash
redis-benchmark -t set -c 100000 -n 100000 -h redis-cluster.redis-cluster.svc.cluster.local
====== SET ======
  100000 requests completed in 8.19 seconds
  100000 parallel clients
  3 bytes payload
12204.05 requests per second
```

### GET
```bash
redis-benchmark -t get -c 200000 -n 200000 -h redis-cluster.redis-cluster.svc.cluster.local
====== GET ======
  200000 requests completed in 15.32 seconds
  200000 parallel clients
  3 bytes payload
  keep alive: 1
13057.39 requests per second
```