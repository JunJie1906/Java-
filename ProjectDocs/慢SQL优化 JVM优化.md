### 慢sql记录：

1、公司企业微信告警

2、DBMS 上 运行SQL，确认是慢sql

3、explain 



#### 关键字优先级优化

1.订单表、用户表

查询某个商品，某个用户购买的订单



原来sql：

```sql
SELECT user.* from
user INNER JOIN order_
on order_.u_id = user.u_id
where order.item_id=x and order_.sale=100
```

order_.u_id 

(order\_.item_id，order\_.sale) 

先执行 join 连表后特别大，而且(order\_.item_id，order\_.sale) 索引没用到



改进：

```sql
SELECT user.* 
from
(
	select * from order_
    where order.item_id=x and order_.sale=100
) t
inner join user on user.uid=order_.uid
```



2.

```sql
SELECT DISTINCT SPUID FROM TEMP_SPMX_JD WHERE SUPPLIER_ID = 'JDSC'
```

执行时间：耗时2.133s

该sql是一个单表查询，有一个联合索引(supplier_id,sku_id) `uk_supplier_id_skuid`，查询优化器在分析各个索引的成本之后，最终选择了这个索引，
为什么这么慢，因为在搜索了索引联合索引(supplier_id,sku_id)的B+树之后，需要根据这个二级索引中叶子节点的id值，继承访问聚簇索引进行回表，因为数据量比较大，所以性能不太好。

因为我们发现，最终返回spuid这一列，而这一列区分度又比较高，所以我们尝试创建联合索引(supplier_id,spuid)，如果有这么一个索引，则spuid不需要再进行回表了，直接访问这个联合索引，就可以返回spuid，这一定要提高性能。

2.3 推荐解决方案
添加索引(supplier_id,spuid)普通B+树索引。

 优化之后执行耗时：0.281s





3.博主线上有一个 cdk 兑换码业务，运营在后台创建一批 cdk 码时，系统会将这批码插入数据库中保存，这样可以保证用户兑换 cdk 时，码在数据库存在才能兑换，保障安全性。当运营创建十万条cdk记录时，线上耗时达到了十几秒。这里用 cdk_info 表举例，表结构展示：

![image-20230808145617539](C:\Users\18110\AppData\Roaming\Typora\typora-user-images\image-20230808145617539.png)



可以看到在单一线程下，插入十万条记录差不多需要15秒了，这十万条数据之间没有关联，互不影响，那我们可以通过线程池提交单一批次的保存任务，配合 CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join() 方法，等所有任务执行完成拿到结果。代码如下：
![image-20230808145700981](C:\Users\18110\AppData\Roaming\Typora\typora-user-images\image-20230808145700981.png)

可以看到执行耗时2.5秒，执行时间缩短了6倍。



