

# 大话RPC











# RPC介绍



RPC主要解决**两个问题**：

1. 解决分布式系统中，服务之间的调用问题；
2. 远程调用时，能够像本地调用一样方便，让调用者感知不到远程调用的逻辑；



不管使用那种协议进行数据传输，一个**完整的RPC过程**如下所示：





![](https://upload-images.jianshu.io/upload_images/7143349-9e00bb104b9e3867.png)

以左边的Client为例：

- Application就是RPC的调用方；
- Client Stub就是代理对象，就是看起来像是Calculator的实现类，其实内部通过RPC进行远程调用的代理对象；
- Client Run-time library就是远程调用的工具包，比如jdk的socket。最后通过底层网络实现数据传输；

过程中最重要的就是**序列化**和**反序列化**；因为传输的数据都是二进制，无法直接发过去一个java对象（对方不认识），因此需要把Java对象序列化为二进制传给Server，Server接收之后再反序列化为java对象；









# 参考

1.[如何解释RPC](https://www.jianshu.com/p/2accc2840a1b) 

