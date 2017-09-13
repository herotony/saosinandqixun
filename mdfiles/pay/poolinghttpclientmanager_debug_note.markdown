### 概述

* 非常值得记录的基于java - httpclient的连接池调试日志。

### 如何判断连接池起效

#### 2017-09-13日志

* 很简单，用Wireshark工具抓包，紧盯http协议看tcp层的source port，如果多次http请求的tcp source port都一样，代表连接池起作用，否则，代表又是新起了一个连接（进一步确认可知一定存在：SYN,SYN+ACK,ACK，三次握手标识来确定一次连接），今天一上午就是发现总是新起连接，还是通过Wireshark工具发现每个连接后来都来自服务端的主动断开请求(FIN+ACK,FIN,ACK)，也就意味着连接池中新建的连接已经断开并被释放。
* 于是查服务端，用**netstat -n |grep tcp-dest-port**，发现一堆TIME_WAIT状态的连接，这是极不正常的!继续查看resin的配置resin.xml，发现如下配置，即**keepalive-time为5秒,而我的测试程序正好是间隔6秒**，导致连接被服务器切断。修改测试程序间隔为1秒，再用Wireshark工具看的确连接被复用了(source port一直不变，在服务端，用netstat -n|grep tcp-dest-port查看的确只有一个连接，不存在大量TIME_WAIT状态)，一切正常！
```xml
                      <server-default>
                                   <jvm-arg>-server</jvm-arg>
                                   <jvm-arg>-Xmx1024M</jvm-arg>
                                   <jvm-arg>-Xms1024M</jvm-arg>
                                   <jvm-arg>-XX:NewSize=512M</jvm-arg>
                                   <jvm-arg>-XX:MaxNewSize=512M</jvm-arg>
                                   <jvm-arg>-XX:SurvivorRatio=8</jvm-arg>
                                   <jvm-arg>-Xss1024k</jvm-arg>
                                   <jvm-arg>-XX:PermSize=256M</jvm-arg>
                                   <jvm-arg>-XX:MaxPermSize=256M</jvm-arg>
                                   <jvm-arg>-XX:+UseConcMarkSweepGC</jvm-arg>
                                   <jvm-arg>-XX:+UseCMSCompactAtFullCollection</jvm-arg>
                                   <jvm-arg>-XX:+CMSParallelRemarkEnabled</jvm-arg>
                                   <jvm-arg>-XX:+CMSClassUnloadingEnabled</jvm-arg>
                                   <jvm-arg>-XX:+PrintGCTimeStamps</jvm-arg>
                                   <jvm-arg>-XX:+PrintGCDetails</jvm-arg>
                                   <jvm-arg>-Xloggc:/home/mdpaygate/logs/resin/gc.log</jvm-arg>
                                   <memory-free-min>1M</memory-free-min>
                                   <thread-max>512</thread-max>
                                   <socket-timeout>15s</socket-timeout>
                                   <keepalive-max>512</keepalive-max>
                                   <keepalive-timeout>5s</keepalive-timeout>
                     </server-default>

```

* 一些常识
  * 现在HTTP 1.1，客户端发起的请求默认就是keep-alive的，即要求复用。
  * Wireshark的使用，很简单，启动后，双击要监控抓包的网卡，然后，过滤HTTP即可看到相应的包信息。
  * TCP的SYN是请求建立连接，FIN是请求断开连接,ACK是确认收到，所以如果来自服务端的FIN，不管如何都会切断连接的
  * 服务端的http keep-alive代表是闲置等待复用时间是5秒，而不是忙着了到5秒就切断，切记！
  * tcp的keepalive不同于http的keepalive，tcp的keepalive定时间隔(interval)发送监测数据包。
  * 完整的PoolingHttpClientManager测试程序在qixunpay项目中存于github中。
