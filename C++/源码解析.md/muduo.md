## Muduo
[连接建立和发送数据](https://mp.weixin.qq.com/s?__biz=MzA3MzU5NDY5Mg==&mid=2648663545&idx=1&sn=4501df315181150dda713176135df077&chksm=872767d2b050eec43cb2f961db71d17a43d75898ddb01c7cff6c2e5e0852cabd89c06cda87e5&token=195924822&lang=zh_CN#rd)

### Acceptor
用于处理新连接，是TcpServer的内部类，数据成员：
- Socket：是Listening socket即server socket
- Channel：用于观察Readable事件，并回调：：handleRead()

Acceptor会进行Socket的创建和bind，这和Tcpserver中的是一样的，并且为Readable设置回调函数readCallback_()这是创建链接时的回调。

Acceptor::listen()会完成listen系统调用并将channel加入到epoll（loop）中。

[Acceptor例子讲解](https://blog.csdn.net/messiran10/article/details/51692078)
