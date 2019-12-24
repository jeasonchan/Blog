# 概述
消费：
即订阅主题，通过while循环的方式实现对订阅的主题的消费，具体的代码实现步骤为：
1.  通过property 实例构造kafka配置实例
2. 通过RetryPolicy、FailSafe结合的方式，再结合所在的消费者group和配置property实例，以多次尝试的方法创建kafka消费者实例
3. 订阅消息主题，主题则必须以collection的数据格式封装好的字符串
4. 通过while循环实现不间断的消息轮询，即poll消息，返回值是ConsumerRecords<K, V>，ConsumerRecords本质就是ConsumerRecord类的List<>集合，常用的就是ConsumerRecords<String, Object>或者ConsumerRecords<String, String>，本质上就是传过来的json串，看反序列化的需求，要反序列化为类实例就是Object，直接当成Map从中取值就是全是String即可，可以使用Google的Gson类进行反序列化。如：
```java
List<Map<String, Object>> ListOfKeyAndValueMap = new Gson().fromJson((String) record.value(), ArrayList.class);
//record是ConsumerRecord<String, Obejct>类型，value()取出来的即为Obejct的集合
//fromJson()的第一个参数要使用String类型的，因此，对record.value()进行类型强转
```


生产：
