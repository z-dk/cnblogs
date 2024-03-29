**key的大小最大上限为512MB**

**value的最大值也是512M**。对于String类型的value值上限为512M，而集合、链表、哈希等key类型，单个元素的value上限也为512M

Redis命令大全:http://doc.redisfans.com/

**登录**

```shell
# -h 主机地址，-p端口号，-a密码
redis-cli -h 127.0.0.1 -p 6379 -a 1234
auth 1234
```

**数据库**

```shell
# 查看所有数据库
config get databases
# 使用1号数据库
select 1
# 设置默认0号数据库
set db_number 0
# 查看键值对的存储结构
object encoding key
```

Redis使用键值对保存数据,每次新增时至少会创建两个对象,一个是键对象,对象共享仅共享(0~9999的字符串)一个值对象,每个对象由**`redisObject`**结构表示:

- **type**:对象类型(REDIS_STRING,REDIS_LIST,REDIS_HASH,REDIS_SET,REDIS_ZSET);用于类型检查,键总是字符串对象
- **encoding**:编码,表明底层实现的数据结构(详细见[数据结构](https://www.cnblogs.com/z-dk/p/17487012.html))
- **ptr**:指向底层实现数据结构的指针
- **refcount**:引用计数,用于内存回收
- **lru**:对象最后一次被命令访问的时间

### String

数据结构(int,embstr,raw)

**int**:long类型的整数,如果执行一些命令而不再是整数,则编码转为raw,比如append追加字符

**embstr**:只读的字符串(无修改程序,需转为raw后再执行修改),长度小于等于39字节,一次内存分配连续空间保存redisObject和sdsstr,如果修改后将转为raw

**raw**:字符串,会调用2次内存分配函数完成redisObject和sdsstr的空间分配;内存释放也一样

1. set key value　　
2. get key 　　
3. del key 　　
4. strlen key 　
5. getset key value 修改键值对
6. getrange key start end获取字符串
7. append key value 将新的字符串加入到原来的字符串

### Hash

数据结构:(**ziplist,hashtable**),使用ziplist的条件:

- 所有键值对(键或值)长度都小于64字节
- 键值对数量小于512个

1. hset key field value　
2. hget key field
3. hdel key field(可以多个删除)
4. hexsists key field
5. hgetall key(获取所有hash结构中的键值对)
6. hkeys key(返回hash中所有的键)
7. hlen key(返回键值对数量)
8. hmset key field value(设置多个键值对)
9. hmget key field(返回多个)　　

### List

数据结构(**ziplist,linkedlist**)使用`ziplist`的条件:

- 元素长度小于64字节
- 元素数量小于512个

1. lpush key node1...(把节点加入到链表左边，对应有rpush)
2. lrange list start end(获取list从start到end的节点值)
3. llen key(获取链表长度)　　

### Set

数据结构(**intset,hashtable**)

使用`intset`的情况:

- 所有元素都是整数
- 元素数量不超过512

set集合是无序的，而且元素不能重复，每一个元素都为string类型

1. sadd key member1...

### Zset

数据结构(**ziplist,skiplist+字典**),其中使用`ziplist`的情况(两个挨着的节点保存一个是成员,一个是分值):

- 每个成员大小小于64字节
- 成员数量小于128个

有序集合，每一个元素多一个score是浮点数，元素唯一,但score可以相等

skiplist结构的zset会使用`字典`保存元素及其分值(用于O(1)查找元素分值),用`跳跃表`完成(ZRANK,ZRANG)操作

1. zadd key score1 value1...　　
2. zrange key start stop(按分值由小到大返回成员) 

### HyperLogLog

基数,是一种算法,用于统计海量数据(无法直接存储或存储开销过大)中非重复数据的数量,比如一本书中存在多少个字符

1. pfadd key element        添加指定元素到hyperLogLog中
2. pfcount key      返回hyperLogLog的基数值
3. pfmerge desKey key1 key2...      合并多个hyperLogLog到desKey中