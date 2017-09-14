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

  * 现在HTTP 1.1，客户端发起的请求默认就是keep-alive的，即要求复用,但可能默认的连接数量就2到3个。
  * Wireshark的使用，很简单，启动后，双击要监控抓包的网卡，然后，过滤HTTP即可看到相应的包信息。
  * TCP的SYN是请求建立连接，FIN是请求断开连接,ACK是确认收到，所以如果来自服务端的FIN，不管如何都会切断连接的
  * 服务端的http keep-alive代表是闲置等待复用时间是5秒，而不是忙着了到5秒就切断，切记！
  * tcp的keepalive不同于http的keepalive，tcp的keepalive定时间隔(interval)发送监测数据包。
  * 完整的PoolingHttpClientManager测试程序在qixunpay项目中存于github中。

* 现在公司(一天目前两百多万单，交易额8000万吧，一个月三十来亿)中午高峰时段，12：00-12：30为150000单，每秒100单好了，那么乘以4倍，就是每秒400单是未来两年的一个设计值，为此，连接池配置连接数为300至400为一个合理区间，多了也就没意义了。如果是4核CPU，则配置400较好，由于是IO密集型，增加服务器比增加CPU核更有效，即来自网卡的接入数量更有效。当然我们配太多也没意义，如果合作方限制了keepalive连接数的话。

### 测试数据

* 配了10个连接，则每秒近10000的情况下，每秒处理四五百，超过1秒的两百多。
* 配了300个连接，则每秒近10000的情况下，每秒处理两千五左右，超过1秒的没有。
* 明天继续写不用连接池的测试，与用连接池做个对比。这个测试用例采用线上版本进行编码。
* 2017-09-14继续测试，线上未采用线程池模式，同等条件下处于ESTABLISHED模式的连接最高630左右，一般130多左右，大量的TIME_WAIT/FIN_WAIT状态，空闲连接不断上升达5千多，甚至更多在上升，如果服务器未设置keepalive timeout，服务器的cpu切换会极其频繁，导致服务器性能下降是必然，为此必须建议设置keepalive timeout，另外，客户端采用http线程池才行，待统计分析数据。10秒完成1700多，最高峰1秒完成七百多。没有过1秒的。对比，昨天的在连接数配置足够多情况下，明显连接池有三倍的性能提高。

* 配置连接池数为600，毕竟最高峰为630，然后再做一次对比看能提高多少，持续一分钟的测试。
#### 连接池数
* 上午先配置了600连接，实际发现经常维持在三四百，结果导致有时600，有时三四百的连接，即运行不到两分钟产生大量的WAIT状态的连接（三千多条），效率反而不如不用连接池，结果是大量的过1秒的，每秒处理能力还不到700，根本原因是连接的关闭不合理导致大量FIN_WAIT/TIME_WAIT状态的连接，随后改成100，则一直维持在100的连接不存在大量WAIT状态的连接，此时每秒处理1700多条，不存在过1秒的记录。
  * 线上高峰期，先用netstat统计WAIT的连接数和ESTABLISHED连接数，再考虑配置的连接池数值，另外，考虑测试关闭连接策略的测试，客户端，服务端都测。
* 不用连接池的数据，每秒500至800，超过1秒白分百，能堆积5000多条WAIT，情况更不堪，算了，不用连接池的测试到此为止。
* 非高峰期线上一台mdpaygate服务器：ESTABLISHED - 最多10  WAIT - 496,那么一台服务器上的连接数配置不能超过100为合适，待定
* 高峰期：ES-26/WAIT -1600
* 下午重点测试连接数配置导致的无效连接被使用问题
  * 将监控线程扫除空闲线程的idle时间调为30秒，好很多，基本没有空闲的啦，idle的监控时长设置太重要了,这部分由于没有将连接数调整为600，测试作废，没参考价值
    * 事实上，600连接下情况极其糟糕，WAIT连接持续增长至6000，还往上...情况甚至远远不如不用连接池，每妙处理几十条，且条条超过1秒。
    * **配置标准是打满保持连接数的情况下，越大越好，配置过大极易导致TIME_WAIT连接**，至于为什么，很奇怪，理论山即便是TIME_WAIT，但总数不该一直涨啊，除非，连接池没取到连接又创建了新的连接。而目前表象就是如此，到此错在哪了...
    * 我又突然明白了，是resin服务端配置了最大512，那么你连接池配600，必然会不断产生88的垃圾连接，一直迅速增长！
