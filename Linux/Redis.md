### Redis

#### 简介

Redis（Remote Dictionary Server）是一个开源的内存数据库，遵守 BSD 协议，它提供了一个高性能的键值（key-value）存储系统，常用于缓存、消息队列、会话存储等应用场景。

-   性能极高：Redis 以其极高的性能而著称，能够支持每秒数十万次的读写操作24。这使得Redis成为处理高并发请求的理想选择，尤其是在需要快速响应的场景中，如缓存、会话管理、排行榜等。
-   丰富的数据类型：Redis 不仅支持基本的键值存储，还提供了丰富的数据类型，包括字符串、列表、集合、哈希表、有序集合等。这些数据类型为开发者提供了灵活的数据操作能力，使得Redis可以适应各种不同的应用场景。
-   原子性操作：Redis 的所有操作都是原子性的，这意味着操作要么完全执行，要么完全不执行。这种特性对于确保数据的一致性和完整性至关重要，尤其是在高并发环境下处理事务时。
-   持久化：Redis 支持数据的持久化，可以将内存中的数据保存到磁盘中，以便在系统重启后恢复数据。这为 Redis 提供了数据安全性，确保数据不会因为系统故障而丢失。
-   支持发布/订阅模式：Redis 内置了发布/订阅模式（Pub/Sub），允许客户端之间通过消息传递进行通信。这使得 Redis 可以作为消息队列和实时数据传输的平台。
-   单线程模型：尽管 Redis 是单线程的，但它通过高效的事件驱动模型来处理并发请求，确保了高性能和低延迟。单线程模型也简化了并发控制的复杂性。
-   主从复制**：**Redis 支持主从复制，可以通过从节点来备份数据或分担读请求，提高数据的可用性和系统的伸缩性。
-   跨平台兼容性：Redis 可以在多种操作系统上运行，包括 Linux、macOS 和 Windows，这使得它能够在不同的技术栈中灵活部署。

#### 安装

```bash
sudo apt-get install redis-server
```

配置文件

```bash
vi /etc/redis/redis.conf
```

查看Redis状态

```bash
sudo systemctl status redis
```

启动Redis服务

```bash
sudo systemctl start redis
```

连接服务器

```bash
redis-cli -h <host> -p <port>

# 连接本地Redis服务
redis-cli
```

#### 配置

```bash
# 获取指定配置项
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME

# 获取所有配置项
redis 127.0.0.1:6379> CONFIG GET *

# 设置指定配置项
redis 127.0.0.1:6379> CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
```

#### 数据类型

Redis 主要支持以下几种数据类型：

-   string（字符串）: 基本的数据存储单元，可以存储字符串、整数或者浮点数。
-   hash（哈希）:一个键值对集合，可以存储多个字段。
-   list（列表）:一个简单的列表，可以存储一系列的字符串元素。
-   set（集合）:一个无序集合，可以存储不重复的字符串元素。
-   zset(sorted set：有序集合): 类似于集合，但是每个元素都有一个分数（score）与之关联。
-   位图（Bitmaps）：基于字符串类型，可以对每个位进行操作。
-   超日志（HyperLogLogs）：用于基数统计，可以估算集合中的唯一元素数量。
-   地理空间（Geospatial）：用于存储地理位置信息。
-   发布/订阅（Pub/Sub）：一种消息通信模式，允许客户端订阅消息通道，并接收发布到该通道的消息。
-   流（Streams）：用于消息队列和日志存储，支持消息的持久化和时间排序。
-   模块（Modules）：Redis 支持动态加载模块，可以扩展 Redis 的功能。

#### 键（key）

-   del key: 该命令用于在 key 存在时删除 key。
-   dump key: 序列化给定 key，并返回被序列化的值。
-   exists key: 检查给定 key 是否存在。
-   expire key seconds: 为给定 key 设置过期时间，以秒计。
-   expireat key timestamp: 类似于 expire，设置 key 的过期时间，接受 UNIX 时间戳。
-   pexpire key milliseconds: 设置 key 的过期时间以毫秒计。
-   pexpireat key milliseconds-timestamp: 设置 key 的过期时间的时间戳，以毫秒计。
-   keys pattern: 查找所有符合给定模式的 key。
-   move key db: 将当前数据库的 key 移动到给定的数据库 db 中。
-   persist key: 移除 key 的过期时间，key 将持久保持。
-   pttl key: 以毫秒为单位返回 key 的剩余的过期时间。
-   ttl key: 以秒为单位，返回给定 key 的剩余生存时间。
-   randomkey: 从当前数据库中随机返回一个 key。
-   rename key newkey: 修改 key 的名称。
-   renamenx key newkey: 仅当 newkey 不存在时，将 key 改名为 newkey。
-   scan cursor [match pattern] [count count]: 迭代数据库中的键。
-   type key: 返回 key 所储存的值的类型。

#### 字符串（String）

```bash
# 设置指定 key 的值，存在则更新
# 返回值：OK
set key value

# 仅在 key 不存在时设置值
# 返回值：设置成功，返回 1。设置失败，返回 0 
setnx key value

# 获取指定 key 的值
# 返回值：key 的值，如果 key 不存在时，返回 nil。如果 key 不是字符串类型，那么返回一个错误
get key

# 获取 key 中字符串值的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)，end 取 -1 代表字符串的最后一位
# 返回值：子字符串
getrange key start end

# 将给定 key 的值设为 value，并返回旧值。如果返回错误，那么也不会设置新值
# 返回值：返回给定 key 的旧值。 当 key 没有旧值时，即 key 不存在时，返回 nil。如果 key 不是字符串类型，那么返回一个错误
getset key value

# 设置或清除指定偏移量上的位，key 不存在的话默认所有位被初始化为 0
# 返回值：指定偏移量原来储存的位
setbit key offset value

# 获取指定偏移量上的位
# 返回值：字符串值指定偏移量上的位(bit)。当偏移量 OFFSET 比字符串值的长度大，或者 key 不存在时，返回 0
getbit key offset

# 同时设置多个 key-value 对
# 返回值：OK
mset key value [key value ...]

# 同时设置多个 key-value 对，只有在所有 key 不存在时
# 返回值：当所有 key 都成功设置，返回 1，否则返回 0
msetnx key value [key value ...]

# 获取所有给定 key 的值
# 返回值：一个包含所有给定 key 的值的列表。如果有某个 key 不存在或者是非字符串类型，那么这个 key 返回 nil 
mget key1 [key2..]

# 用 value 参数覆盖给定 key 的字符串值，覆盖的位置从偏移量 offset 开始，只会覆盖掉与 value 长度一致的区间
# 返回值：被修改后的字符串长度。如果 key 不存在，返回 offset + strlen(value)
setrange key offset value

# 将指定的 value 追加到 key 原来的值末尾。如果 key 不存在， APPEND 就简单地将给定 key 设为 value 
# 返回值：追加指定值后，key 中字符串的长度
append key value

# 返回字符串值的长度
# 返回值：字符串值的长度。当 key 不存在时，返回 0
strlen key

# 将值关联到 key，并设置过期时间
# 返回值：OK
setex key seconds value

# 以毫秒为单位设置 key 的生存时间
# 返回值：OK
psetex key milliseconds value

# 将 key 中数字值增一。如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。本操作的值限制在 64 位(bit)有符号数字表示之内
# 返回值：执行 INCR 命令之后 key 的值。如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误
incr key

# 将值加上给定的增量值。如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 incrby 命令。本操作的值限制在64位有符号数字表示之内
# 返回值：加上指定的增量值之后， key 的值。如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误
incrby key increment

# 将值加上给定的浮点增量值。如果 key 不存在，那么 incrbyfloat 会先将 key 的值设为 0 ，再执行加法操作。key 和 increment可以是指数类型，比如 1e-1
# 返回值：执行命令之后 key 的值
incrbyfloat key increment

# 将 key 中数字值减一。如果 key 不存在，那么 key 的值会先被初始化为 0
# 返回值：执行命令之后 key 的值
decr key

# 将值减去给定的减量值。如果 key 不存在，那么 key 的值会先被初始化为 0
# 返回值：执行命令之后 key 的值
decrby key decrement
```

#### 哈希（Hash）

Redis hash 是一个 string 类型的 field（字段）和 value（值）的映射表，hash 特别适合用于存储对象

Redis 中每个 hash 可以存储 2^32 - 1 键值对（40多亿）

```bash
# 将哈希表 key 中的字段 field 的值设为 value，如果 field 存在则覆盖。如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作
# 返回值：如果字段是哈希表中的一个新建字段，并且值设置成功，返回 1。如果哈希表中域字段已经存在且旧值已被新值覆盖，返回 0 
hset key field value

# 只有在字段 field 不存在时，设置哈希表字段的值
# 返回值：设置成功，返回 1。如果给定字段已经存在且没有操作被执行，返回 0
hsetnx key field value

# 同时将多个 field-value (域-值)对设置到哈希表 key 中。此命令会覆盖哈希表中已存在的字段
# 返回值：OK
hmset key field1 value1 [field2 value2]

# 获取存储在哈希表中指定字段的值
# 返回值：返回给定字段的值。如果给定的字段或 key 不存在时，返回 nil
hget key field

# 获取所有给定字段的值
# 一个包含多个给定字段关联值的表，表值的排列顺序和指定字段的请求顺序一样。如果指定的字段不存在于哈希表，那么返回一个 nil 值
hmget key field1 [field2]

# 获取在哈希表中指定 key 的所有字段和值
# 返回值：以列表形式返回哈希表的字段及字段值。若 key 不存在，返回空列表
hgetall key

# 获取哈希表中字段的数量
# 返回值：哈希表中字段的数量。当 key 不存在时，返回 0
hlen key

# 获取哈希表中的所有字段
# 返回值：包含哈希表中所有域（field）列表。当 key 不存在时，返回一个空列表
hkeys key

# 获取哈希表中所有值
# 一个包含哈希表中所有值的列表。当 key 不存在时，返回一个空表
hvals key

# 删除一个或多个哈希表字段，不存在的字段将被忽略
# 返回值：被成功删除字段的数量，不包括被忽略的字段
hdel key field1 [field2]

# 查看哈希表 key 中，指定的字段是否存在
# 返回值：如果哈希表含有给定字段，返回 1。如果哈希表不含有给定字段，或 key 不存在，返回 0
hexists key field

# 为哈希表 key 中的指定字段的整数值加上增量 increment。如果哈希表的 key 不存在，一个新的哈希表被创建并执行 hincrby 命令。如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 
# 返回值：执行 hincrby 命令之后，哈希表中字段的值
hincrby key field increment

# 为哈希表 key 中的指定字段的浮点数值加上增量 increment
# 返回值：执行 hincrbyfloat 命令之后，哈希表中字段的值
hincrbyfloat key field increment
```

#### 列表（List）

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

一个列表最多可以包含 2^32 - 1 个元素（4294967295, 每个列表超过40亿个元素）

```bash
# 将一个或多个值依次插入到列表头部|尾部。如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作
# 返回值：执行命令后，列表的长度
lpush|rpush key element [element...]

# 将一个值或多个值依次插入到已存在的列表的头部|尾部，列表不存在时操作无效
# 返回值：执行命令后，列表的长度。如果列表不存在，返回 0
lpushx|rpushx key element [element...]

# 获取列表指定范围内的元素，包括 start 和 end
# 返回值：一个列表，包含指定区间内的元素
lrange key start stop

# 移出并获取列表的第一个|最后一个元素
# 返回值：移出的元素。当列表 key 不存在时，返回 nil
lpop|rpop key

# 移出并获取列表的第一个|最后一个元素。如果key1 为空，则获取 key2 的元素，依次下去，直到所有列表没有元素时会阻塞列表，直到等待超时或发现可弹出元素为止
# 返回值：如果列表为空，返回一个 nil。否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值
blpop|brpop key1 [key2] timeout

# 移除列表 source 的最后一个元素，并将该元素添加到列表 destination 的头部并返回
# 返回值：被弹出的元素
rpoplpush source destination

# 移除列表 source 的最后一个元素，并将该元素添加到列表 destination 的头部并返回。如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止
# 返回值：被弹出的元素，超时返回 nil
brpoplpush source destination timeout

# 在列表的元素 pivot 前或者后插入元素 value。当指定元素不存在于列表中时，不执行任何操作。当列表不存在时，被视为空列表，不执行任何操作
# 如果命令执行成功，返回插入操作完成之后，列表的长度。如果没有找到指定元素，返回 -1。如果 key 不存在或为空列表，返回 0
linsert key before|after pivot element

# 根据参数 count 的值，移除列表中与参数 element 相等的元素
# count > 0: 从表头开始向表尾搜索，移除与 element 相等的元素，数量为 count
# count < 0: 从表尾开始向表头搜索，移除与 element 相等的元素，数量为 -count
# count = 0: 移除所有与 element 相等的元素
# 返回值：被移除元素的数量。列表不存在时返回 0
lrem key count element

# 对一个列表进行修剪，保留指定区间内的元素
# 返回值：OK
ltrim key start stop

# 通过索引获取列表中的元素
# 返回值：列表中下标为指定索引值的元素。如果指定索引值不在列表的区间范围内，返回 nil
lindex key index

# 通过索引设置列表元素的值
# 返回值：成功返回OK。当索引参数超出范围，或对一个空列表进行 lset 时，返回一个错误
lset key index element

# 获取列表长度
# 返回值：列表的长度
llen key
```

#### 集合（Set）

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)

集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)

集合对象的编码可以是 intset 或者 hashtable

-   intset：仅用于存储整数值，且在元素数量较少时非常高效。当集合达到一定大小（超过512个元素）时，会自动转换为hashtable，以便提高操作性能
-   hashtable：用于存储字符串或较大的集合，支持更多操作。内存占用相对较高，因为需要存储键值对的哈希表结构，包括指针和哈希桶

```bash
# 向集合添加一个或多个成员，已经存在于集合的成员元素将被忽略
# 返回值：被添加到集合中的新元素的数量，不包括被忽略的元素
sadd key member [member...]

# 获取集合中元素的数量
# 返回值：集合中元素的数量。当集合 key 不存在时，返回 0
scard key

# 返回集合中的所有成员
# 返回值：集合中的所有成员。当集合 key 不存在时，返回空列表
smembers key

# 移除并返回集合中的一个或多个随机元素
# 返回值：被移除的随机元素。 当集合不存在或是空集时，返回 nil
spop key [count]

# 移除集合中一个或多个成员，不存在的成员元素会被忽略
# 返回值：被成功移除的元素的数量，不包括被忽略的元素
srem key member [member...]

# 返回第一个集合与其他集合之间的差异，也可以认为说第一个集合中独有的元素。不存在的集合 key 将视为空集
# 返回值：包含差集成员的列表
sdiff key [key...]

# 将给定集合之间的差集存储在指定的集合中。如果指定的集合 key 已存在，则会被覆盖
# 返回值：结果集中的元素数量
sdiffstore destination key [key...]

# 返回给定所有集合的交集
# 返回值：交集成员的列表
sinter key [key...]

# 返回给定所有集合的交集并存储在 destination 中。如果指定的集合 key 已存在，则会被覆盖
# 返回值：结果集中的元素数量
sinterstore destination key [key...]

# 返回所有给定集合的并集
# 返回值：并集成员的列表
sunion key [key...]

# 所有给定集合的并集存储在 destination 集合中。如果指定的集合 key 已存在，则会被覆盖
# 返回值：结果集中的元素数量
sunionstore destination key [key...]

# 判断 member 元素是否是集合 key 的成员
# 返回值：如果成员元素是集合的成员，返回 1。如果成员元素不是集合的成员，或 key 不存在，返回 0
sismember key member

# 将 member 元素从 source 集合移动到 destination 集合。如果 source 集合不存在或不包含指定的 member 元素，则 SMOVE 命令不执行任何操作。当 destination 集合已经包含 member 元素时， SMOVE 命令只是简单地将 source 集合中的 member 元素删除。
# 返回值：如果成员元素被成功移除，返回 1。如果 member 不是 source 集合的成员，并且没有任何操作对 destination 集合执行，那么返回 0
smove source destination member

# 返回集合中count个随机数
# count > 0，返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合
# count < 0，返回一个长度为-count的数组，数组中的元素可能会重复出现多次
# 返回值：只提供集合 key 参数时，返回一个元素；如果集合为空，返回 nil。如果提供了 count 参数，那么返回一个数组；如果集合为空，返回空数组
srandmember key [count]
```

#### 有序集合（sorted set）

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员

不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序

有序集合的成员是唯一的,但分数(score)却可以重复

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 2^32 - 1 (4294967295, 每个集合可存储40多亿个成员)

```bash
# 向有序集合添加一个或多个成员，或者更新已存在成员的分数。分数值可以是整数值或双精度浮点数
# 返回值：被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员
zadd key score member [score member ...]

# 获取有序集合的成员数
# 返回值：有序集合的成员的数目。当 key 不存在时，返回 0
zcard key

# 返回有序集中，成员的分数值
# 返回值：成员的分数值。如果成员元素不是有序集 key 的成员，或 key 不存在，返回 nil 
zscore key member

# 计算在有序集合中指定分数区间的成员数，包括 min, max
# 返回值：分数值在 min 和 max 之间的成员的数量
zcount key min max

# 在有序集合中计算指定字典区间内成员数量
# 返回值：指定区间内的成员数量
zlexcount key min max

# 通过索引区间返回有序集合指定区间内的成员，其中成员的位置按分数值递增(从小到大)来排序
# 返回值：指定区间内，带有分数值(可选)的有序集成员的列表
zrange key start stop [withscores]

# 返回有序集中指定区间内的成员，通过索引，分数从高到低
# 返回值：指定区间内，带有分数值(可选)的有序集成员的列表
zrevrange key start stop [withscores]

# 通过字典区间返回有序集合的成员
# LIMIT offset count：可选参数，用于限制返回结果的数量。offset 表示跳过前面多少个元素，count 表示返回多少个元素
# 返回值：指定区间内的元素列表
zrangebylex key min max [limit offset count]

# 通过分数返回有序集合指定区间内的成员
# 返回值：指定区间内，带有分数值(可选)的有序集成员的列表
zrangebyscore key min max [withscores] [limit offset count]

# 返回有序集中指定分数区间内的成员，分数从高到低排序
# 返回值：指定区间内，带有分数值(可选)的有序集成员的列表
zrevrangebyscore key max min [withscores] [limit offset count]

# 返回有序集合中指定成员的排名
# 返回值：如果成员是有序集 key 的成员，返回 member 的排名。如果成员不是有序集 key 的成员，返回 nil
zrank key member

# 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
# 返回值：如果成员是有序集 key 的成员，返回 member 的排名。如果成员不是有序集 key 的成员，返回 nil
zrevrank key member

# 移除有序集合中的一个或多个成员，不存在的成员将被忽略
# 返回值：被成功移除的成员的数量，不包括被忽略的成员
zrem key member [member ...]

# 移除有序集合中给定的字典区间的所有成员
# 返回值：被成功移除的成员的数量，不包括被忽略的成员
zremrangebylex key min max

# 移除有序集合中给定的排名区间的所有成员
# 返回值：被移除成员的数量
zremrangebyrank key start stop

# 移除有序集合中给定的分数区间的所有成员
# 返回值：被移除成员的数量
zremrangebyscore key min max

# 有序集合中对指定成员的分数加上增量 increment。当 key 不存在，或分数不是 key 的成员时， ZINCRBY key increment member 等同于 ZADD key increment member
# 返回值：member 成员的新分数值
zincrby key increment member

# 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destination 中。默认情况下，结果集中某个成员的分数值是所有给定集下该成员分数值之和
# numkeys：要计算交集的有序集合的数量
# WEIGHTS weight [weight ...]：指定每个有序集合的权重。默认权重为 1
# AGGREGATE sum|min|max：指定交集结果的聚合方式。默认是 sum（求和）
# 返回值：保存到目标结果集的的成员数量
zinterstore destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]

# 计算给定的一个或多个有序集的并集，并存储在新的 key 中
# 返回值：保存到目标结果集的的成员数量
zunionstore destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
```

#### HyperLogLog

Redis 的 HyperLogLog 是一种用于估算不重复元素数量的数据结构。它特别适合处理大规模数据集，因为它在内存使用上非常高效

-   内存效率：HyperLogLog 使用固定大小的内存（通常为 12 KB），即使在处理数亿个不重复元素时也能保持相对较小的内存占用。

-   近似计数：HyperLogLog 通过 probabilistic algorithms（概率算法）来估算不同元素的数量，因此返回的结果是近似值，而不是精确值。通常，误差在 1% 左右。

-   性能：插入和查询操作都非常快，插入操作的时间复杂度为 O(1

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素

```bash
# 添加指定元素到 HyperLogLog 中，已输入的元素将被忽略
# 返回值：如果至少有个元素被添加返回 1， 否则返回 0
pfadd key element [element ...]

# 返回给定 HyperLogLog 的基数估算值，基数指不重复元素
# 返回值：给定 HyperLogLog 的基数值，如果多个 HyperLogLog 则返回基数估值之和，且基数是不重复的
pfcount key [key ...]

# 将多个 HyperLogLog 合并为一个 HyperLogLog
# 返回值：OK
pfmerge destkey sourcekey [sourcekey ...]
```

#### 发布订阅

Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](./assets/pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](./assets/pubsub2.png)

```bash
# 订阅给定的一个或多个频道的信息，阻塞等待信息
# 返回值：每次接收信息都返回三个值，分别是信息类型，频道，信息
subscribe channel [channel ...]

# 订阅一个或多个符合给定模式的频道
# 返回值：每次接收信息都返回三个值，分别是信息类型，频道，信息
psubscribe pattern [pattern ...]

# 将信息发送到指定的频道
# 返回值：接收到信息的订阅者数量
publish channel message

# 指退订给定的频道，但使用 crtl+c 退出订阅模式，unsubscribe 对于 redis-cli 来说没有实际意义
unsubscribe [channel [channel ...]]

# 退订所有给定模式的频道，punsubscribe 对于 redis-cli 来说没有实际意义
punsubscribe [pattern [pattern ...]]

# 查看订阅与发布系统状态，需要查看对应文档来编写subcommand
# 返回值：对应命令的返回结果
# pubsub channels获取所有活跃频道，即被监听的频道
pubsub subcommand [argument [argument ...]]
```

#### 事务

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

-   批量操作在发送 EXEC 命令前被放入队列缓存
-   收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行
-   在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中

一个事务从开始到执行会经历以下三个阶段：

-   开始事务
-   命令入队
-   执行事务

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的

事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做

```bash
# 标记一个事务块的开始，直到键入exec之前不能重复使用 multi
# 返回值：OK
multi

# 执行所有事务块内的命令
# 返回值：事务块内所有命令的返回值，按命令执行的先后顺序排列。 当操作被打断时，返回空值 nil
exec

# 取消事务，放弃执行事务块内的所有命令，退出 multi，在 exec 执行之前
# 返回值：OK
discard

# 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令（非事务中的命令）所改动，那么事务将被打断。使用 WATCH 监视了一个带过期时间的键，那么即使这个键过期了，事务仍然可以正常执行。watch 在 multi 之前执行
# 返回值：OK
watch key [key ...]

# 取消 watch 命令对所有 key 的监视
# 返回值：OK
unwatch
```

#### 脚本

Redis 脚本使用 Lua 解释器来执行脚本。 Redis 2.6 版本通过内嵌支持 Lua 环境

```bash
# 执行 Lua 脚本
# script： 参数是一段 Lua 5.1 脚本程序。脚本不必(也不应该)定义为一个 Lua 函数
# numkeys： 用于指定键名参数的个数
# key [key ...]： 从 EVAL 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用 1 为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)
# arg [arg ...]： 附加参数，在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)
# e.g.：EVAL "return {redis.call('GET', KEYS[1]), redis.call('GET', KEYS[2]), ARGV[1], ARGV[2], ARGV[3]}" 2 name name1 arg1 arg2 arg3
# 返回值：脚本命令执行后返回结果
eval script numkeys key [key ...] arg [arg ...]

# 将脚本添加到脚本缓存中，但不立即执行该脚本。EVAL 命令也会将脚本添加到脚本缓存中，但是它会立即对输入的脚本进行求值。如果给定的脚本已经在缓存里面了，那么不执行任何操作
# 返回值：给定脚本的 SHA1 校验和
script load script

# 根据给定的 sha1 校验码，执行缓存在服务器中的脚本
# sha1 ： 通过 SCRIPT LOAD 生成的 sha1 校验码
# 返回值：脚本命令执行后返回结果
evalsha sha1 numkeys key [key ...] arg [arg ...]

# 查看指定的脚本是否已经被保存在缓存当中
# 返回值：一个列表，包含 0 和 1，前者表示脚本不存在于缓存，后者表示脚本已经在缓存里面了
script exists sha1 [sha1 ...]

# 从脚本缓存中移除所有脚本
# 返回值：OK
script flush

# 用于杀死当前正在运行的 Lua 脚本，当且仅当这个脚本没有执行过任何写操作时，这个命令才生效
# 返回值：OK
script kill
```

#### 连接

```bash
# 用于检测给定的密码和配置文件中的密码是否相符
# 返回值：密码匹配时返回 OK ，否则返回一个错误
auth [username] password

# 打印字符串
# 返回值：字符串本身
echo message

# 查看服务是否运行
# 返回值：如果连接正常就返回一个 PONG ，否则返回一个连接错误
ping

# 关闭当前连接
quit

# 切换到指定的数据库，数据库索引号 index 用数字值指定，以 0 作为起始索引值。默认使用 0 号数据库
# 返回值：OK
select index
```

#### 数据备份与恢复

```bash
# 创建当前数据库的备份，将在 redis 安装目录中创建dump.rdb文件
save

# 该命令在后台执行
bgsave

# 获取 redis 目录
# 如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可
config get dir
```

#### 安全

```bash
# 获取 config 中的 requirepass 参数
config get requirepass

# 设置 requirepass，客户端连接到 redis 服务的服务
config set requirepass password
```

#### 性能测试

```bash
# 该命令是在 redis 的目录下执行的，而不是 redis 客户端的内部指令
redis-benchmark [option] [option value]

-h		指定服务器主机名，默认为127.0.0.1
-p		指定服务器端口，默认为6379
-s		指定服务器 socket
-c		指定并发连接数，默认为50
-n		指定请求数，默认为10000
-d		以字节的形式指定 SET/GET 值的数据大小，默认为2
-k		1=keep alive，0=reconnect，默认为1
-r		SET/GET/INCR 使用随机 key, SADD 使用随机值
-P		通过管道传输 <numreq> 请求，默认为1
-q		强制退出 redis，仅显示 query/sec 值
--csv	以 CSV 格式输出
-l		生成循环，永久执行测试
-t		仅运行以逗号分隔的测试命令列表
-I		Idle 模式，仅打开 N 个 idle 连接并等待

redis-benchmark -n 10000  -q
```

