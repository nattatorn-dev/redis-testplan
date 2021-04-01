1 master

```
kubectl apply -f 00-namespace.yaml
kubectl apply -f 02-deployment.yaml
kubectl apply -f 02-service.yaml
```


## Forward PORT
kubectl port-forward svc/redis -n redis 10001:6379

## Jmeter Pugins Require
- Redis data set
- Dummy sampler

### Run Redis-cli
```bash
$ redis-cli -h localhost -o 10001
```

```bash
redis-cli -h localhost -p 10001
localhost:10001> LPUSH colors red blue green
(integer) 3
localhost:10001> LRANGE colors 0 -1
1) "green"
2) "blue"
3) "red"
```

## Run Jmeter
```bash
$ jmeter -n -t ./testing/read.jmx -l read-log.jtl
```

## Redis Benchmark
### GET
```bash
$ redis-benchmark -t get -c 10000 -n 100000 -h redis.redis.svc.cluster.local
====== SET ======
  100000 requests completed in 6.64 seconds
  10000 parallel clients
  3 bytes payload
  keep alive: 1
15057.97 requests per second
```

### SET
```bash
$ redis-benchmark -t set -c 10000 -n 100000 -h redis.redis.svc.cluster.local
====== GET ======
  100000 requests completed in 6.55 seconds
  10000 parallel clients
  3 bytes payload
  keep alive: 1
15262.52 requests per second
```


### GET
```bash
$ redis-benchmark -t get -c 100000 -n 100000 -h redis.redis.svc.cluster.local
Could not connect to Redis at redis.redis.svc.cluster.local:6379: Cannot assign requested address
```
### SET
```bash
$ redis-benchmark -t set -c 100000 -n 100000 -h redis.redis.svc.cluster.local
Could not connect to Redis at redis.redis.svc.cluster.local:6379: Cannot assign requested address
```