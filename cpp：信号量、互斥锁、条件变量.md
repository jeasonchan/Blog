# 0 背景
参考文章：

生产者消费者问题   https://www.bilibili.com/video/BV1pg4y1q7Kn/?spm_id_from=trigger_reload

Java通过的对象头中的各种bit位实现了轻量、偏向、重量级等各种锁，CPP中的对象的比较纯碎，没有对象头可以利用，从信号量到互斥量和条件变量，逐渐形成适用于各种场景的"锁"，本文回顾实践一下CPP中的"锁"。

semaphore是C风格的头文件，创建和销毁，和socket一样需要面向过程，比较复杂。mutex和condition_variable则是CPP风格的头文件，稍微好用很多。
