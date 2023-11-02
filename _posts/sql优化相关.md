
#### order by xx limit x的特殊情况
```
where 1=1   and customer_code = ?                                                                         and bill_time = ?                                                                                                                                                                                                                       AND security_bill_status = ?     and is_delete = 0                                                         and id > ?    order by id         limit ?
```
以上为这个sql的查询条件
乍一看貌似没啥问题，但它就是贼慢
使用explain查看后 这个sql的语句为use where，use filesort
filesort 一旦出现就意味着很慢了
经过排除法，定位到问题为`order by id   limit 100 `

问题分析：
1、 order by id，由于主键索引是B+树索引，本身是排好序的。
2、当limit很小时，并且按id排序，mysql的逻辑就会是先排序，再遍历where，查到limit的最大值就直接停止，这种就算法时间就有很大的随机性
   如果总数据为1000w，limit的数据在**前**10w,那预期时间肯定很低，但如果是**后**10w则就时间就倍      
数级增长。

解决方式：
1、把order by id改为order by id+1，这样就变为函数，就不会有限考虑主键排序
2、强制使用索引，可以使其排序完序后再走索引？或者先索引再排序？（这里按后面的想法吧，没现成案例，懒的去实践了）

> [!note] filesort 解释
> 排序时，如果数据量比较小，则在内存（缓冲区）中进行排序，数量大则在磁盘中排序
缓冲区大小、排序字段的数据长度、查询数据条数等都会影响查询性能。


### 强制使用索引

如下，有不走索引的情况

```
SELECT * FROM user u where u.id=100 order by u.update_time
```
于是强制索引
```
SELECT * FROM user u force index(idx_user_id_update_time) where u.id=100 order by u.update_time
```

### Left Join 无法使用索引

A Left Join B on A.ID=B.AID  WHERE A.NAME=? AND B.NAME=?
A表和B表的Name字段都有索引，这里的由于是被左连接B.NAME就不会走索引

### 联合索引-最左前缀匹配原则

最左匹配原则遇到范围查询就停止匹配，如下示例
```
`1 <= id and id <= 3` -> `id in (1,2,3)`
```
