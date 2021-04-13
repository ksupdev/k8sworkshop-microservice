### Note Redis

> Redis is the dictionary/hashmap on remote server (Key-value Store Database)

## steps create redis in K8s

1. Deployment pod redis (1 repicas)
2. Create service สำหรับ access pod redis
3. Create Pod สำหรับทดสอบ connect pod redis ผ่านทาง service โดย pods นี้จะมีการติดตั้ง redis cli

## run workshop

### 0:crate namespace

- crate namespace ``basic-redis``
```powershell
% kubectl apply -f 00-namespace.yml
namespace/basic-redis created

--- Check ---
% kubectl get ns s
NAME              STATUS   AGE
basic-redis       Active   36s
default           Active   58d
ingress-nginx     Active   19h
kube-node-lease   Active   58d
kube-public       Active   58d
kube-system       Active   58d
```

### 1. Deployment pod redis (1 repicas)
- Create redis deployment

> สำหรับ pod ที่ใช้ในการจัดเก็บ data เราควรจะกำหนด replica = 1 เสมอ 

```powershell
% kubectl apply -f 01-deployment.yml
deployment.apps/redis created

--- Check ---
% kubectl get po -n basic-redis
NAME                     READY   STATUS    RESTARTS   AGE
redis-577d58dd6c-x5w5w   1/1     Running   0          95s
```
### 2. Create service สำหรับ access pod redis
- create service
> จะมีการกำหนด clusterIP: None เพื่อที่จะให้เป็น ``Headless Services`` หรือก็คือ service ตัวนี้จะมีการชี้ไปที่ port เดียวเสมอ
```powershell
% kubectl apply -f 02-service.yml
service/redis created
--- Check ---
kubectl get svc -n basic-redis
NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
redis   ClusterIP   None         <none>        6379/TCP   42s
```

### 3. Create Pod สำหรับทดสอบ connect pod redis ผ่านทาง service โดย pods นี้จะมีการติดตั้ง redis cli
- Create pod สำหรับทดสอบ connect redis
```powershell
% kubectl apply -f 03-client-util.yml
pod/client-util created

--- Check ---
% kubectl get pods -n basic-redis
NAME                     READY   STATUS    RESTARTS   AGE
client-util              1/1     Running   0          9m24s
redis-577d58dd6c-x5w5w   1/1     Running   0          25m

```

- Exec into client-util pod ``มันจะเหมือนเราเข้าไปอยู่ใน pod และก็จะสามารถเรียกใช้งาน service ที่อยู่ใน Cluster ได้ทันที``
```powershell
% kubectl exec -it client-util -n basic-redis -- bash
root@client-util:/# 

--- Next step connect redis ---
root@client-util:/# redis-cli -h redis
redis:6379> 

--- test get redis ---
edis:6379> SET "name" "karonn"
OK
redis:6379> GET "name"
"karonn"
redis:6379> 

--- Exit redis
edis:6379> exit
root@client-util:/# 

--- Exit exec
root@client-util:/# exit
exit
karoon@Nuttakorns-MacBook-Pro 01-run-redis %
```

## Redis CLI ()

### client-util pod
- Exec into client-util pod

```powershell
% kubectl exec -it client-util -n basic-redis -- bash
root@client-util:/# 

--- go to redis ----
root@client-util:/# redis-cli -h redis
redis:6379>

--- test set and get ----
redis:6379> SET "myname" "KAROON"
OK
redis:6379> GET myname
"KAROON"

--- Use EXPIRE command to expire mykey in 10 seconds
redis:6379> EXPIRE "myname" 10
(integer) 1

** Wait 10 seconds
redis:6379> GET myname
(nil)

---- DELETE ----
redis:6379> SET "mykey2" "myvalue2"
OK
redis:6379> GET mykey2
"myvalue2"
redis:6379> DEL "mykey2"
(integer) 1
redis:6379> GET mykey2
(nil)

---- List mykey* ----
redis:6379> SET "mykey1" "value1"
OK
redis:6379> SET "mykey2" "value2"
OK
redis:6379> KEYS "mykey*"
1) "mykey2"
2) "mykey1"
redis:6379> 
```

- Cleanup workshop
```powershell
% kubectl delete ns basic-redis
namespace "basic-redis" deleted
% kubectl get pods -n basic-redis
No resources found in basic-redis namespace.
```