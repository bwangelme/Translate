Pub/Sub
=======

SUBSCRIBE，UNSUBSCRIBE 和 PUBLISH 实现了一个消息发布订阅范式，在这个范式中，发送者(publisher)不需要通过程序去发送他们的消息给特定的接收者(subscriber)。相反，发送字符化的消息到频道中，不需要事先知道哪个订阅者(如果存在的话)在频道中。订阅者可能对一个或者多个频道感兴趣，仅仅订阅他们感兴趣的频道就可以了，不需要知道这个频道中有哪些发布者(如果存在的话)。这个在发布者和订阅者之间解耦，可以有更好的可扩展性和更灵活的网络拓扑。
例如为了订阅频道 foo 和 bar，客户端发出了一个 SUBSCRIBE 命令同时提供了频道的名字：

```
SUBSCRIBE foo bar
```

由其他客户端发送到这个频道的消息将会由 redis 推送到订阅了这些频道的客户端。
一个订阅了一个或者多个频道的客户端应该不能发送命令了，尽管它能够从其他频道中订阅或者取消订阅。订阅和取消订阅操作的回复是以消息的形式发送的，所以客户端能够通过消息的第一个元素来知道消息的类型，从而仅仅读相关的消息流。在一个订阅了的客户端的上下文中允许的命令是 SUBSCRIBE, PSUBSCRIBE, UNSUBSCRIBE, PUNSUBSCRIBE, PING 和 QUIT。

## 推送消息的格式

一个消息是一个数组相应包括三个元素：

第一个元素是消息的类型：

> + subscribe: 表示我们成功地订阅了以响应第二个元素命名的频道。第三个参数表示我们当前订阅的频道的数量。
> + unsubscribe: 表示我们成功地取消订阅了以响应第二个元素命名的频道。第三个参数表示我们当前订阅的频道的数量。当最后一个参数为0的时候，表示我们没有再订阅任何频道。当我们在 Pub/Sub 状态之外的时候，客户端可以发送任何类型的 Redis 命令。
> + message: 作为由另外一个客户端发送 PUBLISH 命令的结果，它是一个消息。第二个参数是发起的频道的名字，第三个参数是实际的消息内容。

## 数据库和作用域

Pub/Sub 没有任何相关的键空间，它被设计的在任何等级上都不会被干扰，包括数据库数。
在数据库10上发布的消息，将会被数据库1上的订阅者收到。
如果你需要某种程度上的命名空间，使用环境名称为通道添加前缀(test, staging, production, ...)。

## 有线协议的例子

```
SUBSCRIBE first second
*3
$9
subscribe
$5
first
:1
*3
$9
subscribe
$6
second
:2
```

在此时，我们使用另外一个客户端在名字为 sencond 的频道上执行了一个发布操作。

```
PUBLISH second hello
```

第一个客户端将会接收到：

```
*3
$7
message
$6
second
$5
Hello
```

现在客户端使用 UNSUBSCRIBE 命令(没有额外的参数)取消订阅所有的频道。

```
UNSUBSCRIBE
*3
$11
unsubscribe
$6
second
:1
*3
$11
unsubscribe
$5
first
:0
```