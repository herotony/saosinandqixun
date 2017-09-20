###  简述
* 该文档记录resin公司开发的序列化协议hessian的简版源码复制日志。
* 全部复制在qixunpay项目中。

#### 2017-09-20
* IntMap:简版的hashmap数据结构，估计是用来记录某个方法或url的引用次数。
* HessianLite复制日志
    * HessianRemote:记录远程调用的url的封装对象，实现基本的hashCode()/equal()
    * HessianRemoteResolver:定义了lookup(String type,String url)返回HessianRemote?
    * AbstractHessianResolver:实现lookup -> {return new HessianRemote(type,url);}
    * HessianProtocolException:简单继承自IOException的自定义IOException待后续使用

    * Serializer:定义接口方法writeObject(Object object,AbstractHessianOutput out) ，这里暂时定义个空类:AbstractHessianOutput
    * Deserializer:定义反序列化接口，同样暂时定义空类：AbstractHessianInput

    * AbstractSerilizerFactory:抽象类，仅仅定义了getSerializer(Class c1)/getDeserializer(Class c1)两个接口分别获取Serializer/Deserializer接口对象。
    * AbstractSerializer:简单以抽象方式实现Serializer接口的writeObject方法。
    * SerializerFactory：实现AbstractSerializerFactory，**暂时空类,最后才能完整实现**

    * AbstractHessianInput:与output组建关键接口功能。
    * AbstractHessianOutput:定义好该接口，很关键的类，多看看!

    * BasicSerializer:实现AbstractSerializer的writeObject方法。
