# 面试题
## 1. Kafka 是如何保证消息不丢失的？
首先 Kafka 对消息不丢失有两个必要条件，一是对于发送给 broker 确认过的消息，Kafka 对消息的持久性做出了保证，二是对于保存消息的多个 broker 只要
有一个还活着，消息就能保证不丢失。下面从消息的生产者和 broker 来说明 Kafka 是如何保证消息不丢失的。消息的生产者我们可以在发送消息的时候使用send
加回调函数的方式，对于由于网络原因导致消息发送不成功的情况，设置一个比较大的重试次数，保证避免网络原因导致消息丢失，acks 设置成 all，保证所有的broker
都确认过消息。对于 broker 端，保证消息不丢失的方式是先消费消息后更新 offset。

参考资料：https://cloud.tencent.com/developer/article/1589157

## 2. Kafka 是如何保证消息不被重复消费？
生产者端可以设置幂等，由 broker 处理生产者发送消息的幂等，但这种方式只能保证单个分区幂等，消费者保证不重复消费消息，有两种方式一是去重，可以使用
roaringbitmap，二是在业务侧做幂等。

参考资料：https://cloud.tencent.com/developer/article/1625607
