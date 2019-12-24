# 杂七杂八
postman中，param就是key value就是path中的键值对，也就是restfu中的queryParam，就是传给resource方法的输入参数。headers和body中的都有键值对，分别是什么？如何使用postman发送一个文件/流？
答：
restful中，发送的请求有：url（ip+port+path)+key/value+header(状态码、body描述)+body（json串、文件流），返回的东西是header+body。
header中的key、value正是描述body中将要发送的东西的描述。比如，如果想在body中放json串，那么header中的Content-Type对应的值就是application/json，对应的body中选择raw、json就可以发送json串，然后服务端resource方法接收到body发过来的json串，自动反序列化为map或者POJO/Bean。
同理，如果要发送文件，header的key：Content-Type对应的值就要换成流，表示以字节流的形式发送文件，对应的，body中，选择form-data，还要在key位置选择发送file还是text，value就是将要发送的文件，发送文件时，服务端resources方法接收文件本身的流和文件的头部信息。

---


# mock server步骤：
0、必须先建立一个collection，再点collection上的三角形，切换到mock标签页，建立环境/服务器，用于存储将要造出来的假数据，只需要设置假服务器名字即可，比如“mock serer”，已经有环境了就不用再创建了，或者自己想拥有多个也可以再创建；（同样的，也可以使用左上角“+”号新建mock server，也可以通过右上角环境名称旁边的齿轮添加）
1、postman的假接口其实基于原先的请求接口来制造假数据的，所以先把真实的请求接口写好，或者直接复制一份原先的真实的请求，将请求放到上面建立的collection中，url地址先不改，因为url地址之后可能要变成假服务器的地址
2、在这个真实的请求的页面中，点example，add example，填写response的body和header，也就是返回的假数据，注意右上角的environment的名字，比如“mock server”，这些假数据就是保存到这个“mock server”上的
3、基于假服务器“mock server”的假数据建立完毕，这个environment的服务器是互联网可见的，要多加注意被攻击的可能
4、返回修改原请求中的url地址，地址换为假服务器“mock server”的地址，假服务器url的查看方式：collection→小三角形→mock标签，即可查看mock服务器的名称和对应的url地址
