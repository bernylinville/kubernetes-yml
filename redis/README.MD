# redis

```bash
# k8s-master
mkdir -p /data/k8s/redis/pv{1..6}

cat /etc/exports
/data/k8s/redis/pv1 192.168.50.0/24(rw,sync,no_root_squash)
/data/k8s/redis/pv2 192.168.50.0/24(rw,sync,no_root_squash)
/data/k8s/redis/pv3 192.168.50.0/24(rw,sync,no_root_squash)
/data/k8s/redis/pv4 192.168.50.0/24(rw,sync,no_root_squash)
/data/k8s/redis/pv5 192.168.50.0/24(rw,sync,no_root_squash)
/data/k8s/redis/pv6 192.168.50.0/24(rw,sync,no_root_squash)

systemctl restart rpcbind
systemctl restart nfs
systemctl enable nfs

kubectl apply -f redis/pv.yml
persistentvolume/redis-nfs-pv1 created
persistentvolume/redis-nfs-pv2 created
persistentvolume/redis-nfs-pv3 created
persistentvolume/redis-nfs-pv4 created
persistentvolume/redis-nfs-pv5 created
persistentvolume/redis-nfs-pv6 created

kubectl create configmap redis-conf --from-file=redis/redis.conf --namespace uatfop

kubectl describe cm -n uatfop redis-conf
Name:         redis-conf
Namespace:    uatfop
Labels:       <none>
Annotations:  <none>

Data
====
redis.conf:
----
appendonly yes
cluster-enabled yes
cluster-config-file /var/lib/redis/nodes.conf
cluster-node-timeout 5000
dir /var/lib/redis
port 6379
Events:  <none>

kubectl apply -f redis/service.yml
service/redis-service created

kubectl get svc -n uatfop redis-service
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
redis-service   ClusterIP   None         <none>        6379/TCP   11s

kubectl apply -f redis/redis.yml
statefulset.apps/redis-app created

[root@k8s-master kubernetes-yml]# kubectl get pod -n uatfop -o wide |awk '{print $6}'
IP
10.244.28.81
10.244.59.74
10.244.28.82
10.244.59.75
10.244.28.83
10.244.59.76

yum -y install redis-trib

redis-trib create --replicas 1 \
10.244.28.81:6379 \
10.244.59.74:6379 \
10.244.28.82:6379 \
10.244.59.75:6379 \
10.244.28.83:6379 \
10.244.59.76:6379

# 或者采用部署redis-trib容器初始化：参考链接https://github.com/iNem0o/docker-redis-trib

kubectl exec -it -n uatfop redis-app-0 -- redis-cli -c
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_sent:276
cluster_stats_messages_received:276
127.0.0.1:6379> role
1) "master"
2) (integer) 85
3) 1) 1) "10.244.59.75"
      2) "6379"
      3) "85"


```
