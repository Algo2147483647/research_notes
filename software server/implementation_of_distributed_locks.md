# Implementation of distributed locks

[TOC]

## Distributed lock base on redis

### SET NX EX/PX + Lua

**Lock**

```sh
SET lock_name random value EX PX $time
```

**Unlock**: Execute Lua script to verify random value when releasing the lock

```sh
eval "if redis.call("get",KEYS[1]) == ARGV[1] then return redis.call("del",KEYS[1]) else return 0 end"
```

- 通过随机数, 保证只解锁自己的那把锁. 避免因为超时, 自己的锁被超时释放了, 自己解锁的时候导致解了别人的锁
- PX 保证 EX + PX 是一个原子操作, 避免并发冲突
- SET NX 实现分布式锁功能, 利用 Redis 作为分布式锁服务