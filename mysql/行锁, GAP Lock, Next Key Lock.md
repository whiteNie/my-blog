### 行锁

### 行锁升级为表锁

索引失效(rows 超过总行数的30%)的情况下会升级为表锁

### 常见的 X LOCK

* record lock 给指定行记录添加锁
* gap lock 间隙锁
* next key lock

