## Redisson分布式锁

Redisson是Java版的Redis客户端

Java分布式锁不仅可以在同一台机器的多线程状态下使用，还可以在分布式系统的多台机器下使用。它保证了线程无法接触到上锁了的资源。





```
/**
 * 尝试获取锁
 * @param lockKey
 * @param unit 时间单位
 * @param waitTime 最多等待时间
 * @param leaseTime 上锁后自动释放锁时间
 * @return
 */
public  boolean tryLock(String lockKey, TimeUnit unit, int waitTime, int leaseTime) {
    RLock lock = redissonClient.getLock(lockKey);
    try {
        return lock.tryLock(waitTime, leaseTime, unit);
    } catch (InterruptedException e) {
        return false;
    }
}
```


