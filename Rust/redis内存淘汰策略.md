# redis内存淘汰策略

```
redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "4294967296"  (显示的是字节数，4 * 1024 * 1024 * 1024)

127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
```

当 Redis 达到 maxmemory 上限时，它需要知道如何“腾出”空间以存入新的数据。这就是 maxmemory-policy 的作用。如果不设置这个策略，当内存满时，Redis 将对所有写入操作返回错误，服务将变为只读状态。

# 设置 `maxmemory-policy` 内存淘汰策略

当 Redis 达到 `maxmemory` 上限时，它需要知道如何“腾出”空间以存入新的数据。这就是 `maxmemory-policy` 的作用。**如果不设置这个策略，当内存满时，Redis 将对所有写入操作返回错误，服务将变为只读状态。**

在文件中找到 `# maxmemory-policy noeviction` 这一行，去掉注释并选择一个适合您业务场景的策略。

**常用的策略包括：**

- `noeviction`: (默认值) 内存满后，任何写入操作都会返回错误。不删除任何数据。
- `allkeys-lru`: **(非常常用)** 从所有键 (keys) 中，移除最近最少使用的 (LRU - Least Recently Used)。
- `volatile-lru`: 只从设置了过期时间的键中，移除最近最少使用的。
- `allkeys-random`: 从所有键中，随机移除一个。
- `volatile-random`: 只从设置了过期时间的键中，随机移除一个。
- `allkeys-lfu`: **(Redis 4.0+ 新增)** 从所有键中，移除最不经常使用的 (LFU - Least Frequently Used)。对于很多场景比 LRU 更高效。
- `volatile-lfu`: 只从设置了过期时间的键中，移除最不经常使用的。
- `volatile-ttl`: 只从设置了过期时间的键中，移除最接近过期时间的。

**示例：**
选择 `allkeys-lru` 作为淘汰策略。

```
# redis.conf

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select one of the following behaviors:
#
# volatile-lru -> Evict using approximated LRU among keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among keys with an expire set.
# allkeys-random -> Remove a random key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.

maxmemory-policy allkeys-lru
```