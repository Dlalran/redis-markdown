# Redis

​		官方网站：[Redis](https://redis.io/)，[GitHub-Redis](https://github.com/antirez/redis)

​		Redis是开源免费、遵守BSD协议、使用ANSI C语言编程、支持网络、可基于内存也可持久化的非关系型(NoSQL)的Key-Value数据库，其提供了多种语言的API。

​		键值(Key-Value)存储类型的NoSQL数据库主要使用一个哈希表，表中的键和指针指向特定的数据。其优点是可以使得查询速度快、存放数据量大且支持高并发，但缺点是存储的数据缺乏结构、不支持复杂的条件查询。

​		Redis的优势是性能高、丰富的数据类型、原子性、丰富的特性、高速读写，缺点是持久化代价大和占用内存高。

​		常用的应用场景有：缓存、分布式会话(Session)、分布式锁、消息系统、排行榜、计数器等

---

## 安装与启动

#### 安装

##### 安装包方式

​		Redis默认提供基于Linux环境的安装，从[下载页](https://redis.io/download)获取安装包。Redis是基于C语言开发的，因此需要预先安装GCC环境。

1. 通过`gcc -v`检查是否已安装GCC环境，若没有则安装

```sh
yum -y install gcc automake autoconf libtool make
```

2. 下载安装包，也可以通过上传获得

```sh
wget http://download.redis.io/releases/redis-[版本号].tar.gz
```

3. 解压缩安装包(这里解压缩到`/opt`目录)

```sh
tar -xzvf redis-[版本号].tar.gz -C /opt
```

4. 进入目录并通过Makefike文件编译并构建可执行程序

```sh
cd redis-[版本号]
make
```

5. 安装到`/usr/local/redis`目录中

```sh
make PREFIX=/usr/local/redis install
```

##### Docker方式

1. 从Docker Hub拉取Redis镜像

```
docker pull redis:[版本号]
```

2. 创建并运行Redis容器

   ​		先在宿主机目录创建`redis.conf`，并挂载该文件到容器以实现自定义配置启动，也可以在运行参数中直接指定(如`--requirepass "12345"`)。还可以将持久化数据文件目录(`dir`配置指定位置，其自动创建的位于`/data`)进行数据卷挂载以获得数据库数据。

```
 docker run -d -p 6379:6379 -v [宿主机文件目录]:/usr/local/etc/redis/redis.conf --name [容器名] redis:[版本号] redis-server [运行参数] /usr/local/etc/redis/redis.conf
```

3. 通过`redis-cli`访问服务端

```
docker exec -it [容器名] redis-cli
```



#### 启动与关闭

##### 服务端启动

​		进入安装目录的bin文件夹并运行`redis-server`，查看Redis服务端是否启动成功

```sh
cd /usr/local/redis/bin
./redis-server
```

​		*可以通过Ctrl + C退出Redis服务端进程，并输入`bg`令其后台继续运行，也可以在运行时指定命令为`./redis-server &`来令其后台运行，还可以在Redis配置文件`redis.conf`中设置`daemonize yes`*

##### 服务端关闭

1. 强制关闭

   ​		通过`ps -ef | grep -i redis`查询Redis进程PID，并通过`kill -9 [PID]`杀死进程。

2. 正常关闭

   ​		启动并登录到客户端后，在客户端输入`shutdown`即可关闭服务端，该方式会自动保存未持久化的数据。

##### 客户端启动

1. 可以通过`./redis-cli`直接运行Redis客户端，完整启动命令如下，不指定时默认为本机IP且端口为6379

```sh
redis-cli -h [服务器地址] -p [端口号] -a [服务端密码]
```

2. 通过`ping`命令检查和服务端是否联通，显示`PONG`则联通正常

3. 测试通过Key-Value方式存取值，此处key为foo，value为bar

```
set foo bar
get foo
```

##### 客户端关闭

​		通过命令`exit`退出客户端。

---

## 配置

#### 自定义配置

​		如果需要自定义配置，需要把解压目录的配置文件复制到安装目录下

```sh
cp /opt/redis-[版本号]/redis.conf /usr/local/redis
```

​		启动时在命令中指定配置文件所在的路径

```sh
./redis-server ../redis.conf
```



#### 配置项

​		仅列出重要配置项，完整配置见[Redis配置](https://www.runoob.com/redis/redis-conf.html)

- `daemonize`：Redis是否以守护进程方式运行(即后台运行)，默认为`no`，开启则修改为`yes`
- `pidfile`：Redis服务端运行时将对应进程pid写入的文件路径，默认为`pidfile /var/run/redis_6379.pid`
- `port`：监听端口号，默认为`6379`
- `bind`：指定绑定的主机地址，即发向本机该地址的(指定网卡地址)接受的请求才会被Redis接受，默认为回环地址`127.0.0.1`，即只接受本机请求，**需要接受外部访问时，将该配置注释，或指定为Redis所在服务器网卡地址，或者`0.0.0.0`(即当前服务器所有网卡地址)**
- `protected-node`：保护模式，需要设置`bind`或者设置密码才能通过外网访问
- `timeout`：客户端闲置多长时间后连接将会被断开，默认为`300`(单位为秒)
- `loglevel`：日志级别，总共有四种：`debug`(大量信息，适用于开发、测试)、`verbose`(略微精简的信息)、`notice`(适量信息，适用于生产环境)、`warning`(仅严重警告)，默认为`verbose`
- `databases`：设置数据库的个数，默认为`16`，使用命令`SELECT [数据库ID]`来切换数据库，默认使用的数据库是0
- `save [时间间隔] [修改次数]`：指定在指定时间间隔内有多少次更新操作，就将数据同步到持久化的数据文件，可以指定多个该条件，默认为`save 900 1`(900秒内1个更改)、`save 300 10`(300秒内10次更改)、`save 60 10000`(60秒内10000次更改)
- `rdbcompression`：对数据持久化时是否进行压缩处理，采用LZF压缩算法，默认是`yes`
- `dbfilename`：指定本地持久化数据库文件名，默认为`dump.rdb`
- `dir`：指定本地持久化数据库文件存放路径，默认为`./`，即`redis-server`所在目录中
- `slaveof [master地址] [master端口]`：设置本机为指定服务器的从属服务器，当该服务器启动时，将从主服务器(master)进行数据同步
- `masterauth`：当主服务器(master)设置了密码保护时，指定连接的密码
- `requirepass`：设置服务端连接密码，客户端连接后通过`auth [密码]`来进行登录
- `maxclients`：设置同一时间最大客户端连接数
- `maxmemory`：设置最大内存限制，单位是KB
- `maxmemory-policy`：指定超过最大内存限制时的淘汰策略，详细见下

---

## 内存维护

​		Redis中经常存储大量的数据，因此需要按照一定的策略对内存进行即时整理，以维护系统性能。

- 为key设置有效时间

  ​		默认所有的键值对都永久有效(即有效时间为-1)。

  1. 通用的设置Key有效时间方式

``` 
expire [Key] [有效时间(秒为单位)]
```

​		     2. String类型数据设置方式，同时完成Key与Value的关联以及设置Key的有效时间

```
setex [Key] [有效时间] [Value]
```

- 采用LRU算法进行淘汰

  ​		当占用内存超过指定值(`maxmemory`)时，Redis提供的淘汰策略有如下

  1. `volatile-lru`：清除设置了有效时间且最不常使用的数据 (**常用**)
  2. `allkeys-lru`：清除所有key中最不常用的数据 (**常用**)
  3. `volatile-lfu`：清除设置了有效时间前使用频率最少的数据
  4. `allkeys-lfu`：清除所有key中使用频率最少的数据
  5. `volatile-random`：随机清除设置了有效时间的数据
  6. `allkeys-random`：随机清除所有key中的数据
  7. `volatile-ttl`：清除最早过期(当前有效时间最小)的数据
  8. `noeviction `：不做任何处理，不允许写入，仅允许读取

---

## Redis命令

​		Redis命令是在Redis客户端上运行的，用于对Redis服务上执行操作。

#### Key管理

​			Key长度尽量不要超过1024B，区分大小写，一个项目中最好采用统一的命名模式(通过`:`进行分割)，Value存值最大为512M。

- `keys [patterns]`：查看符合给定模式的所有key，模式通配符有 * (代表所有)，? (代表任意一个字符)
- `exists [key]`：是否存在指定的key，存在则返回1，否则返回0
- `set [key] [value]`：设置key-value对，即将指定value关联到key
- `get [key]` ：获取指定key的value
- `del [key]`：删除指定key
- `expire [key] [有效时间]`：对指定key设置有效时间，单位为秒，命令为`pexpire`则为毫秒
- `ttl [key]`：查询指定key的剩余有效时间，不存在或已过期返回-2，永久有效则返回-1。同上有`pttl`
- `persist key`：设置指定key为永久有效
- `move [key] [数据库ID]`：将当前数据的key转移到其他数据库
- `randomkey`：随机返回一个key
- `rename [key] [new_key]`：将指定key重命名为new_key
- `type [key]`：返回指定key关联的value类型

#### 数据库管理

​		Redis下的数据库是通过一个整数来进行标识，数据库间相互独立，可以在配置文件设置数据库的数量(见配置项说明)，客户端默认连接到数据库0。

- `info`：查看当前数据库信息
- `select [数据库id]`：客户端切换数据库
- `dbsize`：查看当前数据库key数量
- `move [key] [数据库id]`：移动key到指定数据库
- `flushdb`：清空当前数据库
- `flushall`：清空所有数据库

#### 其他命令

- `ping`：测试和服务端的连通性，连通正常时不带参数时返回`PONG`，带参数时返回原参数
- `exit`：退出客户端
- `shutdown [save|nosave]`：关闭当前连接的服务端，参数指定保存或不保存
- `save`：阻塞形式(使用任务进程，不建议使用)对数据库内数据进行持久化，`bgsave`为非阻塞形式(后台异步，启用新进程)持久化，持久化目录以及方式见配置
- `config get [参数名]`：获取指定配置参数值，`config set [参数名] [参数值]`：修改指定配置值
- `monitor`：实时输出Redis服务端收到的命令请求

---

## 数据类型

​		Redis支持的常用五大数据类型为：String(字符串)、Hash(哈希)、List(列表)、Set(集合)、Zset(有序集合)。

### 基本数据类型

#### String类型

​		String类型是Redis最基本的数据类型，该类型Value最大能够存储数据长度为512MB。该类型是二进制安全的，即可以存储任何数据。

**赋值命令**

- `set [key] [value]`：普通的赋值语法，若该key已存在则覆盖其value，无视类型
- `setnx [key] [value]`：如果key不存在则赋值并返回1，存在则不赋值且返回0（分布式锁的实现方案之一）
- `setex [key] [有效时间] [value]`：设置key-value对，并设置其有效时间
- `setrange [key] [开始位置] [替换值]`： 将指定key的value从指定位置开始替换为指定值(从0开始,下同)
- `mset [key1] [value1] [key2] [value2] ...`：批量赋值

**取值命令** 

- `get [key]`：普通的取值语法，不存在则返回nil，如果value不是String类型则返回错误
- `getrange [key] [开始位置] [结束位置]`：获取指定key的value的指定范围的字符串(包含首尾)
- `getbit [位置]`：获取指定位置上的值
- `getset [key] [value]`：使用指定value覆盖指定key的值，并返回原来的value，不存在该key则返回nil
- `strlen [key]`：获取指定key的value长度
- `mget [key1] [key2] ...`：批量取值

**其他命令**

- `incr [key]`：将指定key中存储的value加一，如果不存在，则创建一个value为0的指定key，并加一。如果指定key的value不为数字类型String，则返回错误
- `incrby [key] [增量]`：将指定key存储的value增加指定值，`incrbyfloat`为增加浮点值
- `decr [key]`、`decrby [key] [减量]`：递减，同上
- `append [key] [value]`：将指定key的value末尾追加指定值，不存在则设值

​		*删除命令`del [value]`适用于所有类型数据。*



#### Hash类型

​		Hash类型是String类型的Field和Value的映射表，可以看成具有Key和Value的Map容器。其适合于存储对象，相比于以JSON方式存储在String类型中，Hash类型占用更少的内存空间，并且省去序列化和反序列化的开销。每个Hash类型可以存储$${2}^{32}-1$$个键值对。

**赋值命令**

- `hset [key] [field] [value]`：对指定的key，设定field-value，一个key表示一条数据(对象)，field表示数据的一个属性，value则是该属性对应的值
- `hsetnx [key] [field] [value]`：当指定key内的指定field不存在时，才新增field-value
- `hmset [key] [field1] [value1] [field2] [value2] ...`：对指定key批量新增field-value
- `hincrby [key] [field] [增量]`为指定key的指定field的value增加指定增量，`hincrbyfloat`为增加浮点值

**取值命令**

- `hget [key] [field]`：获取指定key内指定field的值
- `hmget [key] [field1] [field2] ...`：批量获取指定key内指定field的值
- `hgetall [key]`：获取指定key内所有的field-value
- `hkeys [key]`：获取指定key内的所有field
- `hvals [key]`：获取指定key内所有field的value
- `hlen [key]`：获取指定key内field的数量

**删除命令**

- `hdel [key] [field1] [field2] ...`：删除指定key内的一个或多个field

**其他命令**

- `hexists [key] [field]`：查看指定key中是否存在指定field



#### List类型

​		List类型是简单的字符串列表，按照插入的顺序排序，可以添加元素到列表头或列表尾。一个列表最多可以包含$${2}^{32}-1$$个元素。常用于对数据量大的数据进行删减、任务队列等场景。

**赋值命令**

- `lpush [key] [value1] [value2] ...`：将一个或多个value插入到指定列表头部(左侧)
- `rpush [key] [value1] [value2] ...`：将一个或多个value插入到指定列表尾部(右侧)
- `lpushx [key] [value]`：将一个值插入到已存在列表的头部，若不存在则取消操作
- `rpushx [key] [value]`：将一个值插入到已存在列表的尾部，若不存在则取消操作
- `lset [key] [位置] [value]`：替换列表中指定位置元素值为给定元素
- `linsert [key] before/after [元素值] [value]`：在列表的<u>指定值元素</u>前/后插入给定元素

**取值命令**

- `llen [key]`：获取指定列表的长度
- `lindex [key] [位置] `：获取列表对应位置的元素
- `lrange key [开始位置] [结束位置]`：获取列表指定范围内的元素 （从0开始，负数下标表示倒数第几个元素）

**删除命令**

- `lpop [key]`：移除并返回列表头元素
- `rpop [key]`：移除并返回列表尾元素
- `brpop [key1] [key2] ... [等待时间]`：按指定key顺序查看对应的list，弹出第一个非空列表的尾元素，并返回弹出的列表key和弹出的元素值，如果都为空则等待(阻塞)指定时间，若仍无法弹出则返回nil。同理`blpop`为头元素。
- `lrem [key] [数量] [value]`：数量 > 0时，删除从表头开始指定数量等于指定值的元素，数量 <  0时从表尾开始删除，数量 = 0时删除所有等于指定值的元素。
- `ltrim [key] [开始位置] [结束位置]`：删除列表中不在指定范围内的所有元素

**其他命令**

- `rpoplpush [源列表key] [目标列表key] `：移除源列表的表尾元素，并将该元素放入目标列表表头中并返回
- `brpoplpush [源列表key] [目标列表key] [等待时间]`：同上，如果源列表中无元素则等待指定时间。



#### Set类型

​		Set类型是String类型的无序集合，集合的元素必须是唯一的，因此集合中不允许出现重复的数据。Redis中Set类型是通过来HashTable实现的，一个集合最多可以包含$${2}^{32}-1$$个元素。可以对其唯一性和集合的运算来进行应用。

**赋值命令**

- `sadd [key] [value1] [value2] ...`：向指定集合添加一个或多个成员

**取值命令**

- `scard [key]`：获得指定集合成员数
- `smembers [key]`：获得集合的所有成员
- `sismember [key] [value] `：判断元素是否为集合的成员(集合中是否存在对应元素)
- `srandmember [key] [count]`：随机返回集合中指定数量的成员

**删除命令**

- `srem [key] [value1] [value2] ...`：删除集合中一个或多个指定成员
- `spop [key] [count]`：随机删除并返回集合中指定数量的成员
- `smove [源集合key] [目标集合key] [value]`：将源集合的指定成员移动到目的集合中

**集合运算命令**

- `sdiff [key1] [key2]`：返回集合间的差集
- `sdiffstore [目标集合key] [key1] [key2]`：返回集合一和集合二的差集并存储在目标集合中，下同省略
- `sinter [key1] [key2]`：返回集合间的交集
- `sunion [key1] [key2]`：返回集合间的并集



#### ZSet类型

​		ZSet(Sorted Set)类型是String类型的有序集合，也不允许有重复的成员。其顺序是通过对每个成员关联一个double类型的数字(score)实现的，根据分数的递增排序来对集合成员进行排序，分数允许重复。可以应用于排行榜等相关的应用场景。

**赋值命令**

- `zadd [key] [score1] [value1] [score2] [value2]`：向指定有序集合添加一个或多个成员，并指定或更新其对应的分数

**取值命令**

- `zcard [key]`：获取指定有序集合的成员数
- `zcount [key] [起始分数] [终止分数]`：获得有序集合中指定分数区间内的成员数，获得所有分数区间则起始和终止分数分别设为`-inf`、`+inf`，可以通过`(分数值`和`[分数值`来指定开闭区间(默认为闭区间)，下同
- `zrank [key] [value]`：获得有序集合中指定成员的分数
- `zrange [key] [开始位置] [结束位置] [WITHSCORES]`：按照分数升序返回有序集合中指定位置区间的所有成员，`WITHSCORES	`指定是否返回成员对应的分数。`zrevrange`为按照分数降序
- `zrangebyscore [key] [起始分数] [终止分数] [WITHSCORES] [LIMIT offset count]`：根据分数升序返回有序集合中指定分数区间的成员，最后一组参数指定返回结果的偏移数量和数量(可用于分页显示，如`LIMIT 4 2`可以看作返回每页两个结果的第三页)。`zrevrangebyscore`为按照分数降序

**删除命令**

- `zrem [key] [value]`：删除指定有序集合中一个或多个成员
- `zremrangebyrank [key] [开始位置] [结束位置]`：删除有序集合中按照分数升序指定位置区间的所有成员
- `zremrangebyscore [key] [起始分数] [终止分数]`：删除有序集合中指定分数区间的所有成员

**其他命令**

- `zincrby [key] [增量] [value]`：对有序集合中指定元素的分数增加指定增量，并返回增加后的分数



### 高级数据类型

​		Redis中的高级数据类型都是为了解决单一问题而存在的，因此使用场景较小，复杂性也较低。

#### Bitmaps

​		Bitmaps其实并不是一种数据类型，而是String类型的一种按位操作的接口，可以看作是一个每位只能存储0或1的数组。

​		一般将Bitmaps的位数作为标识(如用户的id)，该位的值作为数据。在这种情况下，如果标识并不从0开始(如用户id从10000开始)，可以统一减去一个偏移量再进行存储，以避免初始化和存取值时的性能低下。

**操作命令**

- `setbit [key] [偏移量] [value]`：设置Bitmaps第偏移量位上值为指定值，偏移量位前未设置的位值将全部设为0
- `getbit [key] [偏移量]`：获取Bitmaps指定位上的值，不存在时也会返回0
- `bitcount [起始位] [终止位]`：获取Bitmaps指定位区间内值为1的位个数
- `bitop [位操作] [目标key] [源key1] [源key2] ...`：对源key按位进行位操作(包括and、or、not、xor)，并将结果存放到目标key中



#### HyperLogLog

​		HyperoLogLog是Redis用于基数(集合中不重复元素的数量)统计的算法类型，其通过输入的元素来计算对应的基数，而不会存储输入的元素本身。

​		每个HyperLogLog只需要花费12 KB内存，就可以计算接近$${2}^{64}$$个不同元素的基数。因此一般用于对数据量大的数据集进行基数估算，例如对访问用户数量统计等应用场景。

**操作命令**

- `pfadd [key] [element1] [element2] ...`：输入元素到指定HyperLogLog中
- `pfcount [key1] [key2] ...`：返回一个或多个指定HyperLogLog的基数估计值
- `pfmerge [目标key] [源key1] [源key2] ...`：将多个源HyperLogLog合并到目标HyperLogLog中



#### GEO

​		GEO数据类型是Redis提供地理位置支持的数据结构，实际其内部实现是ZSet类型。

**操作命令**

- `geoadd [key] [经度 纬度 位置名] ...`：存储一个或多个的空间元素(经度、纬度和位置名)到指定key中
- `geopos [key] [位置名] ...`：获取指定key中所有给定位置名的经纬度
- `geodist [key] [位置名1] [位置名2]`：获取给定两个位置之间的距离
- `georadius [key] [经度] [纬度] [半径] [单位] [参数] [排序方式] [COUNT 数量]`：返回指定key中离给定经纬度中心点，距离在给定半径以内的位置名；单位有`m`、`km`、`mi`和`ft`；参数有`WITHDIST`(附加距离)、`WITHCOORD`(附加经纬度)、`WITHHASH`(附加GeoHash编码的字符串)；排序方式有距离递增`ASC`和距离递减`DESC`；还可以通过`COUNT n `来只获取前n个符合的结果。
- `georadiusbymember [key] [位置名] [半径] [单位] [参数]`：基本同上一命令，除了以指定key内已存在的位置点作为中心点，而不是另外给定中心点的经纬度。

---

## 连接Redis

#### Jedis

​		Jedis是Redis在Java的实现客户端，提供了较为全面的Redis操作特性。其使用阻塞的I/O和同步的方法调用，因此不是线程安全的。需要通过连接池来使用。

##### 添加依赖

​		通过Spring Boot集成Jedis不需要指定版本号。

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

##### 配置参数

​		配置Redis和Jedis连接池的参数，详细见注释。

```yml
spring:
  redis:
#    Redis服务端主机地址
    host: 192.168.2.128
#    服务端端口号
    port: 6379
#    服务端连接密码
    password: 12345
#    连接超时时间(ms)
    timeout: 2000
    jedis:
#      Jedis连接池配置
      pool:
#        最大连接数
        max-active: 10
#        最大空闲连接数
        max-idle: 6
#        最小空闲连接数
        min-idle: 2
#        最大阻塞等待时间
        max-wait: 10000

```

##### 配置类

​		在配置类中获取上面的配置参数并封装到`JedisPoolConfig`中，并通过其创建并注入Jedis连接池类`JedisPool`。

```java
@Configuration
public class JedisConfig {
    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Value("${spring.redis.password}")
    private String password;

    @Value("${spring.redis.timeout}")
    private int timeout;

    @Value("${spring.redis.jedis.pool.max-active}")
    private int maxActive;

    @Value("${spring.redis.jedis.pool.max-idle}")
    private int maxIdle;

    @Value("${spring.redis.jedis.pool.min-idle}")
    private int minIdle;

    @Value("${spring.redis.jedis.pool.max-wait}")
    private int maxWait;

    @Bean
    public JedisPool jedisPool() {
        /**
         * 注入Jedis连接池
         */
//        封装配置信息，不配置则为默认值
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(maxActive);
        jedisPoolConfig.setMaxIdle(maxIdle);
        jedisPoolConfig.setMinIdle(minIdle);
        jedisPoolConfig.setMaxWaitMillis(maxWait);
//        设置连接耗尽时的行为，true为阻塞直至超时(默认)，false为报异常
        jedisPoolConfig.setBlockWhenExhausted(true);

        return new JedisPool(jedisPoolConfig, host, port, timeout, password);
    }
}
```

##### 操作Redis数据

​		Jedis中的方法和上述Redis命令相对应，通过`JedisPool`获取Jedis连接，调用方法来执行对应命令，使用后关闭连接。

​		**注意:实体类要实现序列化接口`Serializable`,且属性名前两位必须全为小写或全为大写**

- 操作String类型

  ​		下面先在Redis中查询是否存在指定key，若存在则通过Fastjson将JSON格式字符串反序列化为对象。如果不存在则将对象User序列化为json格式字符串并放入Redis String类型数据中(**需要引入Fastjson依赖**)。

  ```java
      public User testRedisString(String id) {
  //        通过连接池获取Jedis对象
          Jedis jedis = jedisPool.getResource();
          String key = "user:" + id;
  
          User user = null;
  //        判断key是否存在
          if (jedis.exists(key)) {
              String value = jedis.get(key);
  //            反序列化为对象，如果对象类型包含泛型则需要使用TypeReference
              user = JSON.parseObject(value, User.class);
          } else {
  //            模拟数据库查询
              user = new User("1", "Howard", 22);
  //            通过Fastjson将对象转换为json格式字符串
              String value = JSON.toJSONString(user);
  //            将该key-value放入Redis中
              jedis.set(key, value);
          }
  
  //        注意使用后关闭连接
          jedis.close();
          return user;
      }
  ```

- 操作Hash类型

  ​		逻辑同上，但是Hash类型的操作都基于Map类型对象，因此需要手动将对象转换为Map或者通过Fastjson进行转换，这里的存放格式是通过key来区分不同对象，field为属性名，value为属性值。

  ​		*此处通过Fastjson在对对象和Map<String, String>的相互变换过程中进行了重复的序列化和反序列化操作，不建议这样使用，应当通过BeanUtils工具或者手动方式完成相互转换。*

  ```java
  public User testRedisHash(String id) {
          String key = "user:" + id;
          Jedis jedis = jedisPool.getResource();
          User user = null;
  
          if (jedis.exists(key)) {
              Map<String, String> value = jedis.hgetAll(key);
  //            Map -> JSON字符串 -> JSON对象
              JSONObject jsonObject = JSON.parseObject(JSON.toJSONString(value));
  //            JSON对象 -> 对象
              user =(User) jsonObject.toJavaObject(User.class);
          } else {
              user = new User("2", "John", 23);
  //            将对象序列化为JSON对象(Object -> JSON)
              JSONObject jsonObject = (JSONObject) JSONObject.toJSON(user);
  //            将JSON对象反序列化成Map(JSON -> Map)
  //            注意：此处泛型对象的反序列化需要使用TypeReference
              Map<String, String> map = jsonObject.toJavaObject(new TypeReference<Map<String, String>>(){});
  //            放入Redis Hash中(Map -> field-value)
              jedis.hmset(key, map);
          }
  
          jedis.close();
          return user;
      }
  ```



#### Lettuce

​		Lettuce是基于Netty的Redis客户端实现，其可以在多个线程并发访问，并线程安全，同时采用可伸缩设计，连接实例不够时可以按需增加连接实例。

​		这里使用的是Spring Data Redis，其中对Redis底层开发包(Jedis、Lettuce)进行了封装，并通过RedisTemplate提供对于Redis的操作、异常处理和序列化，对同一类型的数据封装为Operation并自动管理连接池。

##### 添加依赖

​		Spring Boot 2.0+中添加Spring Data Redis启动器，其中默认集成Lettuce，同时还要添加其依赖的Commons-pool.

```xml
<!--        Redis(Lettuce)-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
<!--        Commmons-pool(Lettuce依赖需要)-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
```

##### 配置参数

​		基本和Jedis相同，区别是设置后不用手动装配，`RedisProperties`会自动装配。

```yml
spring:
  redis:
    #    Redis服务端主机地址
    host: 192.168.2.128
    #    服务端端口号
    port: 6379
    #    服务端连接密码
    password: 12345
#    连接超时时间(ms)
    timeout: 2000
    lettuce:
      pool:
#        最大连接数(负值为无限制)
        max-active: 8
#        最大空闲连接
        max-idle: 8
#        最小空闲连接
        min-idle: 0
#        最大阻塞等待时间(负值为无限制)
        max-wait: 1000
```

##### 配置类

​		注入自定义RedisTemplate，指定连接实现、序列化方式等配置。

```java
@Configuration
public class RedisConfig {
    /**
     * @Description Redis配置类
     */
//    注入RedisTemplate，指定使用Lettuce客户端，并在其中做其他配置
    @Bean
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
//        使用Lettuce连接Redis
        template.setConnectionFactory(factory);

//        指定Key的序列化方式
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringRedisSerializer);
        template.setHashKeySerializer(stringRedisSerializer);
        

//        使用Jackson序列化方式代替默认的JDK序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
//        配置自定义ObjectMapper
        ObjectMapper objectMapper = new ObjectMapper();
//        序列化的范围(all包括field、getter、setter、creator)，和可见性修饰符(any包括public、private等)
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
//        序列化的类型必须是非final的
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
//        指定Value的序列化方式
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
//        注意根据需要来对String类型的value进行序列化，如果在String类型中存储对象JSON字符串则使用Jackson序列化方式
        template.setValueSerializer(stringRedisSerializer);
        
//        初始化RedisTemplate
        template.afterPropertiesSet();
        return template;
    }
}
```

##### 操作Redis数据

​		`RestTemplate`对不同类型的数据操作封装为不同的Operations，调用这些Operation的方法来边界的操作Redis。

- 操作String类型	

```java
//    RedisTemplate操作String类型
    public String testString(String key) {
        String result = null;
//        查询是否存在key，等同于exists
        if (redisTemplate.hasKey(key)) {
//            opsForValue封装对于String类型的操作
            result = (String) redisTemplate.opsForValue().get(key);
        } else {
            result = "ValueContent";
            redisTemplate.opsForValue().set(key, result);
        }
        return result;
    }

//    设置String的有效时间
    public void testExpireString(String key, String value) {
        redisTemplate.opsForValue().set(key, value);
//        作为不限制数据类型的命令expire，不被封装到任何数据类型的operation中
//        参数依次指定key、有效时间值、有效时间单位
        redisTemplate.expire(key,10, TimeUnit.SECONDS);
        log.info("设置有效时间为10秒的数据key:" + key + "，value:" + value);
    }
```

- 操作Hash类型

  ​		`RestTemplate`会对对象进行配置中指定方式的序列化，存储方式是指定key和field(这里是hashkey，用于区分不同对象)，将对象的属性作为value放入Hash类型中。**注意与Jedis的区别**

```java
   public User selectById(String id) {
        User result = null;
//        相当于Redis命令 hget user id
        if (redisTemplate.opsForHash().hasKey("user", id)) {
            redisTemplate.opsForHash().get("user", id);
        } else {
            result = new User("1", "howard", 22);
            redisTemplate.opsForHash().put("user", id, result);
        }
        return result;
    }
```

- 操作List类型

```java
    public void listTest1(ArrayList<String> messages) {
        String key = "msg";

//        将字符串一一放入list中
//        messages.forEach(message -> listOps.leftPush(key, message));
        listOps.leftPushAll(key, messages);

//        删除指定key的第count个值为value的元素，count>0时，删除从表头开始count个等于value的元素，count<0时从表尾开始删除，count=0时删除所有等于value的元素
        listOps.remove(key, 0, messages.get(1));

//        获取list中所有元素
        List<String> results = listOps.range(key, 0, -1);
        results.forEach(System.out::println);
    }
```

- 优化

  ​		可以将数据类型的封装操作Operations进行统一注入代替手动获取；同时对泛型进行指定，代替手动类型转换。

  ​		具体实现是通过`@Resource`注解，再通过Spring的属性编辑器(PropertyEditor)机制，在不同Operations的Editor(如`HashOperationsEditor`)中的方法`setValue`中，通过`RedisTemplate`获得对应的Operations并注入。

```java
@Resource(name = "redisTemplate")
private HashOperations<String, String, User> hashOps;

 public User selectById(String id) {
        User result = null;
        if (hashOps.hasKey("user", id)) {
            result = hashOps.get("user", id);
        } else {
            result = new User("1", "howard", 22);
            hashOps.put("user", id, result);
        }
        return result;
    }
```

---

## 订阅

​		Redis发布订阅是一种消息通信模式，发送者(pub)发送消息，订阅者(sub)接受消息，客户端可以订阅(subscribe)任意数量的频道(channel)。当有新消息发送到频道时，这个消息就会被发送给订阅该频道的所有客户端。

#### 命令

1. 订阅频道

- `subscribe [频道] ...`：订阅一个或多个频道的消息
- `psubscribe [模式] ...`：订阅一个或多个符合给定模式的频道

2. 发布频道

- `publish [频道] [消息]`：将消息发送到指定的频道

3. 退订频道

- `unsubscribe [频道] ...`：退订指定的频道
- `punsubsrcribe [模式] ...`：退订指定模式的所有频道

---

## 事务

​		在事务中可以一次按顺序串行执行多个指令，执行过程不会被其他客户端提交的命令插入。事务中有命令出现语法错误时，事务终止输入并放弃；如果出现数据类型操作不正确等执行后才会报告的错误，则除错误命令外的其他命令正常执行。Redis事务并不支持回滚。

#### 命令

- `multi`：标记一个事务的开始，事务内的指令在执行该指令后一一给出，这些指令会被加入到事务队列中
- `discard`：取消当前事务，放弃已输入的所有指令
- `exec`：结束事务指令输入并执行当前事务
- `watch [key] ...`：监视一个或多个key，在`multi`命令前使用，当事务执行前这些key被其他客户端所改动，则取消事务的执行(即执行`exec`命令后会返回nil)，是Redis对于乐观锁的一种实现。
- `unwatch`：取消对于所有key的监视

---

## 主从复制

#### 概念

​		主从复制指的是master(主服务端)的数据即时有效的复制到slave(从服务端)中，其中master主要提供数据写功能(也可以提供读)，执行写操作后将变化的数据同步到slave中，slave仅提供读操作以及备份功能。

**作用**

​		其作用是实现读写分离、负载均衡、故障恢复与数据冗余，基于主从复制构建哨兵模式和集群，从而实现Redis高可用。

**三个阶段**

- 建立连接阶段

  ​		首先slave中设置master的地址和端口并发送ping，master响应后根据信息创建连接的socket；slave周期性地发送ping命令，master作出响应以确定连接正常；如果master设置了密码，slave发送密码以进行验证授权；master中再保存slave的端口号，主从完成连接。

- 数据同步阶段

  ​		slave向master发起同步请求(如执行`psync2`指令)，master收到后进行后台持久化(`bgsave`指令)来生成rdb文件；当第一个slave连接时，master会创建复制积压缓冲区(即aof文件)来保存写操作；rdb文件生成完毕后通过socket发送给slave，slave接受后清空数据并执行通过rdb文件恢复数据过程，该过程也称为**全量复制**。

- 命令传播阶段

  ​		master间歇性发送命令缓冲区内的指令，slave收到后通过aof数据恢复方式(`bgrewriteof`指令)进行数据恢复，master端中需要维护每个slave端当前数据同步的偏移量，对每个slave仅发送其未同步的部分指令，该过程也称为**部分复制**。

  ​        每次部分复制时slave对master id和偏移量进行检查，如果master的id发生变化或者出现偏移量小于master存储的偏移量情况，则进行全量复制。

**注意事项**

- 对于部分复制，master的复制积压缓冲区的设置要充足，否则当下一次部分复制还未进行时，缓冲区已经发生溢出，则被迫进行全量复制，甚至陷入死循环，通过`repl-backing-size [缓冲区大小]`来进行设置，建议设置为$$2*\overline{主从间重连时长}*\overline{每秒写命令数据总量}$$。
- 如果对数据一致性要求较高，可以在slave进行主从复制时关闭其对外服务，配置`slave-serve-stale-data no`后，slave进行主从复制时，将会对外部请求进行阻塞，完成后再进行处理。
- 一个master拥有过多slave时注意调整拓扑结构，如从一主多从调整为树状结构，从而降低master的同步压力。但是也要注意，树状结构层次过深时，底层与顶层的服务端数据同步延迟较大，数据一致性变差，因此需要均衡调整。
- master关闭时执行`shutdown save`来进行持久化，id和偏移量将都被存储到rdb文件中，重启时将这两个值读取会内存中，这样可以防止slave在master重启后进行全量复制。

**心跳机制**

​		命令传播阶段中，master和slave之间需要不断进行信息交换，因此使用心跳机制保证双方连接在线。

- master在一定间隔内(由`repl-ping-slave-period`决定，默认为10秒)ping所有的slave来判断slave是否在线，通过`info replication`可以查看所有slave的状态。
- slave每隔1秒通过`replconf ack [偏移量]`来向master报告当前数据同步偏移量，同时也确认master在线。master发现偏移量不同时会发起部分复制，同时确认slaves的数量和各自的延迟。
- master可以配置`min-slaves-to-write [最小slaves数]`和`min-slaves-max-log [最大延迟数]`，当slave数量小于指定值或者所有slave的延迟均大于指定值时，强制关闭写功能，并停止数据同步。



#### 连接方式

- 客户端`redis-cli`中输入命令`slaveof [master ip] [master port]`

- slave服务端启动时加入参数`redis-server -slaveof [master ip] [master port]`

- 配置文件中修改配置项`slaveof [master ip] [master port]`

  slave端通过`slaveof no one`来断开连接

---

## 哨兵

#### 概念

​		哨兵(Sentinel)是不提供数据服务的Redis服务端，用于对主从结构中的每台服务器进行监控，一般设置为数量为单数的分布式系统，当master宕机时选择新的master并重新调整主从结构。

**监控阶段**

- Sentinel指定master并启动后，会向master发出`info`指令获取master当前状态、slaves数量和状态，并建立cmd连接，在sentinel和master两端维护并同步包含以上信息以及sentinel列表的信息表，然后再通过获得的slave列表向各个slave发送`info`指令确认各自状态。
- 有新sentinel加入后，会在master的sentinel列表中感知到还存在其他的sentinel，更新该列表后ping其他的sentinel，并互相订阅发布的消息，从而形成一个sentinel网络。

**通知阶段**

- 每个sentinel都会对master和salve通过cmd连接发送指令来进行监控，并将实时状态发布给其他的sentinel

**故障转移**

- 当sentinel多次向master发送指令未收到回复时，其会认为其主观下线(`sdown`)，并通知其他的sentinel去向该master发出指令
- 当超过配置中能够判断master宕机的sentinel数值(见下`sentinel monitor`项`quonum`参数)都认为该master宕机，则认为其状态为客观下线(`odown`)
- 确定客观下线后，需要从一众sentinel中选出leader来进行故障切换工作，采用的是[Raft算法](https://www.cnblogs.com/xybaby/p/10124083.html)，每个sentinel都向其他sentinel发出选举信息，每个sentinel都会选举先收到其消息的sentinel，当一个sentinel获得的票数超过`quonum`且超过sentinels总数的一半时，该sentinel被选举为leader并通知其他sentinel。如果不能达到则间隔一段时间后重新发起选举。
- 该sentinel从是否在线、和master断开连接的时长、slave优先级(`slave-priority`指定，值越小优先级越高)、同步数据的偏移量大小(越大越好)、运行id(选择runid较小的)依次考察每个slave，并从中选择出新的master
- 选出新master后，向新master执行`slaveof no one`使其成为master，并设置其为其他slave的master，更新所有sentinel以及相关的状态表，最终完成故障切换



#### 搭建

- 设置sentinel的配置文件`sentinel.conf`，示例如下

```
port 26379
proteceted-mode no

# 配置监控的master，quonum为判定master宕机需要的sentinel数量，一般设置为哨兵总数一半的向上取整
sentinel monitor [master名] [master ip] [master port] [quonum]
# 配置master的密码
sentinel auth-pass [master名] [master密码]
# 判断master宕机的超时时间
sentinel down-after-miiliseconds [master名] [毫秒数]
# 主从切换后最多slave同时同步数量
sentinel parallel-syncs [数量]
# 整个故障切换的超时时间(超过时间将取消同步数量限制)，以及下次故障切换的间隔时间
sentinel failover-timeout [master名] [毫秒数]
```

- 启动时通过`redis-sentinel`来启动sentinel，如`redis-sentinel sentinel.conf ` 
- master宕机后sentinel将自动完成整个故障切换过程，可以在sentinel的日志中查看整个过程，其中会包括监测到master下线、通知其他sentinel、重新选举master、更新监控master、调整主从结构和下线原master等[详细过程](https://www.cnblogs.com/moonandstar08/p/5001902.html)。

---

## 集群

#### 概念

​		Redis官方的集群方案Redis Cluster集群采用无中心结构，至少各有三个主服务端(master,主要提供写操作)和三个从服务端(slave，只提供读操作)才能建立建立集群。每个节点可以是一台机器的个不同端口、或者是不同的机器。每个节点保存数据和整个集群的状态，节点间两两连接，客户端仅连接其中任意一个可用节点。

**可用性**

​		节点的可用性采用投票机制，如果半数以上主节点与该主节点通信超时，则认为该主节点当前不可用(fail)，其从节点将替代其成为主节点，如果无从节点则整个集群不可用。如果半数以上的主节点不可用，则无论是否有从节点，该集群不可用。

**节点分配**

​		在集群模式下，所有的key根据哈希函数映射到0-16383个整数槽(slot)内，每个节点负责维护一部分槽以及槽所映射的key-value数据。



#### 集群搭建

​		以下给出Redis 5+版本的集群搭建过程示例。

- 创建集群目录，如`/usr/local/redis_cluster`，在目录下建立每个节点的配置文件目录，如以端口号命名7000-7005
- 复制redis配置文件`redis.conf`到各个目录下，并对配置文件做以下修改，创建一个配置文件后，其他节点只需要在编辑器下输入`:%s/[原端口号]/[新端口号]/g`将端口号替换即可

```
port [端口号]
# 开启集群模式
cluster-enabled yes
# 节点配置文件名，该文件由系统管理
cluster-config-file nodes-[端口号].conf
# 节点超时时间
cluster-node-timeout 5000
daemonize yes
# 进程id存储文件
pidfile /var/run/redis_7000.pid
# 日志文件名
logfile "7000.log"
#bind  127.0.0.1
# 设置连接主节点master的密码
masterauth [主节点密码]
# 设置本节点密码，与主节点保持一致
requirepass [本节点密码]
```

- 将Redis安装目录中包含`redis-server`所需可执行文件的`src`目录复制到集群目录中，并通过它启动各个节点

```sh
cp -r /opt/redis-[版本号]/src /usr/local/redis_cluster/
./src/redis-server ./[端口号]/redis.conf
```

​		还可以编写脚本文件`startup.sh`来一次性启动，在其中列出每一个节点的启动命令。

- Redis 5+通过`redis-cli`来直接创建集群，通过以下命令，其中指定主节点的密码、每个节点的地址以及端口号，最后一个参数指定主从的复制比，本示例中是3主3从，因此为1

```sh
./src/redis-cli --cluster create -a [主节点密码] 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```

​		运行后会显示每个节点分配的slots，节点间的主从关系等信息，输入`yes`确认后将按照该配置建立集群



#### 集群使用

​		Redis集群的构造是去中心化的，因此客户端只需要连接集群中的任意一个节点就可以获取所有节点中的数据。

- 通过`redis-cli`连接到集群节点，注意添加参数`-c`以集群模式连接

```sh
./src/redis-cli -c -h [节点地址] -p [节点端口] -a [节点密码]
```

- 通过命令查看集群相关信息
  - 通过`info replication`查看本节点主从信息
  - 通过`cluster info`查看集群的大概信息
  - 通过`cluster nodes`查看整个集群节点信息，节点间通过id来唯一标识

- 如果读写操作时数据在另一个节点中存储，将会显示提示信息，包含数据key映射的槽位和存储的节点地址



#### 集群关闭

​		可以通过`redis-cli`连接每个节点，并通过命令`shutdown`来关闭节点，也可以通过以下命令直接关闭

```
[目录]/redis-cli -c -h [节点地址] -p [节点端口] -a [节点密码] shutdown
```

​		还可以创建一个脚本`shutdown.sh`，将每个节点的关闭命令列出，从而一次性关闭所有节点。创建后通过命令`chmod u+x shutdown.sh`来使其变为可执行文件。



#### 节点变更

**新增节点**

- 先通过上述方法配置并启动新节点，连接原有集群的节点并通过`cluster nodes`查看新节点是否启动成功，记录该节点的id
- 通过`redis-cli --cluster reshard [集群任意节点地址]:[端口号]`来重新分配该节点所在集群的slot，输入需要迁移的slot数量，并输入新节点的id
- 选择迁移策略，输入`all`则将从所有节点分配对应的slot给新节点，否则手动输入所有分配slot的源节点id，并以`done`结束
- 通过`cluster nodes`查看新节点是否已经加入集群以及分配的slot

**新增从节点**

- 配置并启动新节点，但并不给其分配slot
- 通过以下命令将其作为某个节点的从节点加入集群

```
redis-cli --cluster add-node [新节点地址]:[端口号] [集群任意节点地址]:[端口号] --cluster-slave --cluster-master-id [主节点id]
```

**删除节点**

- 如果是从节点，则直接通过`redis-cli --cluster del-node [节点地址]:[端口号] 节点id`来删除
- 如果是主节点，
  - 可以为其加入一个从节点，然后直接将其删除，系统将通过故障切换让从节点进行顶替。
  - 先查询该节点被分配的slot，再通过`redis-cli --cluster reshard [节点地址]:[端口号]`将该节点的slot重新分配到其他节点中，再进行删除。

---

## 性能监测与测试

#### 性能测试

​		Redis自带了一个叫redis-benchmark的工具来模拟一定数量的客户端同时对服务端发出一定数量的请求，并显示测试结果参数，从而达到对服务端的性能、以及网络连接进行测试的效果。

##### 使用

​		redis-benchmark工具在Redis的`bin`目录下，通过命令`./redis-benchmark [参数]`来对指定服务端进行性能测试，常用参数有`-h [主机名] -p [端口名] -a [密码] -c [客户端数(并发数)] -n [请求数] -q(静默模式测试，仅显示结果)`



#### 性能监测

​		通过第三方工具可以对Redis的性能进行实时监测，例如采用[Redis_exporter](https://github.com/oliver006/redis_exporter)监控工具配合监控告警系统[Prometheus](https://github.com/prometheus/prometheus)，再通过数据度量可视化工具[Grafana](https://github.com/grafana/grafana)对监控采集的性能数据进行显示。

​		以下给出基于Docker的三种工具安装与部署方式简易示例。

##### Redis_exporter

- 通过docker拉取镜像`docker pull oliver006/redis_exporter`
- 通过参数指定redis服务端地址与密码并运行容器`docker run --name redis_exporter -d -p 9121:9121 oliver006/redis_exporter --redis.addr redis://[redis服务端地址]:6379 --redis.password '[密码]' `
- 访问http://localhost:9121/metrics，查看是否能够显示Redis的性能参数

##### Prometheus

- 通过docker拉取镜像`docker pull prom/prometheus`
- 创建并编写Prometheus配置文件`prometheus.yml`，示例如下

```yml
global:
# Prometheus抓取数据间隔
  scrape_interval: 15s

# 采集目标
scrape_configs:
  - job_name: redis_exporter
    static_configs:
      - targets: ['[Redis_exporter所在主机地址]:9121']
        labels:
          instance: redis_exporter
```

- 创建并运行容器`docker run --name prom --hostname prom -p 9090:9090 -d -v [prometheus.yml目录]:/etc/prometheus/prometheus.yml prom/prometheus`

- 访问http://localhost:9090/targets，查看Redis_exporter端点是否成功运行并被监测

##### Grafana

- 通过docker拉取镜像`docker pull grafana/grafana`
- 创建其用于存储数据的数据卷文件夹，如`/usr/local/grafana`，并通过`chmod 777`开放该目录其他用户读写权限
- 创建并运行容器`docker run -d -p 3000:3000 --name grafana -v /usr/local/grafana:/var/lib/grafana grafana/grafana`
- 访问http://localhost:3000，用户名和密码默认是admin，点击Configuration - Data Sources - Add data source - Prometheus，输入Prometheus的地址并保存
- 在官网下载用于Prometheus Redis的可视化监控台(Dashboard)模板，例如id为[763](https://grafana.com/grafana/dashboards/763)的模板json，然后在Grafana中的Create - Import导入该json文件，选择创建好的数据源，即可查看Redis的性能监控数据
