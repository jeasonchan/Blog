# 1 前言

参考文章：

重复读取InputStream的方法    https://blog.csdn.net/u014656173/article/details/77280705


进行MyBatis学习时，SqlSessionFactoryBuilder需要使用转换为流的配置文件。

但是，我将配置文件  getResourceAsStream  后，先readAllBytes，将配置文件打印了出来，然后再给SqlSessionFactoryBuilder构造对象。但是，报了EOF的错误，应该不是文件末尾的地方，却读到Byte标志位——文件末尾，报了该异常。

这个异常让我想起了，writeObject和readObject方法，向输出流中连续写入两个对象，但是读的时候尝试读三个对象，在第三次调用readObject时就会报错，同样的EOF异常。

再结合，NIO中的ByteBuffer中的书签、pos、limit的机制，推测InputStream也有类似的pos变量，用来标记读到到哪个字节了。

下面解析一下的为什么InputStream不能重复读取，和如何实现对InputStream的重复读取。

# 2 为什么设计为不能重复读取

“InputStream就类比成一个杯子，杯子里的水就像InputStream里的数据，你把杯子里的水拿出来了，杯子的水就没有了，InputStream也是同样的道理。”
比喻的非常好，让我们从直观上认识了InputStream为什么不能重复被读。

从更深的代码角度去分析：

在InputStream读取的时候，会有一个pos指针（position指针），他指示每次读取之后**下一次要读取的起始位置**，当读到最后一个字符的时候，pos指针不会重置,而是指向一个不存在的位置。

看一下几种InputStream的read方法里的关于pos移动的代码片段：

```java
//BufferedInputStream代码片段：  
 public synchronized int read() throws IOException {  
        if (pos >= count) {  
            fill();  
            if (pos >= count)  
                return -1;  
        }  
        return getBufIfOpen()[pos++] & 0xff;  
    }  

//FileInputStream代码片段：  
public native int read() throws IOException;
```

Java 的List内部是使用数组实现的，遍历的时候也有一个pos指针。但是没有说List遍历一个第二次遍历就没有了，List是可以重复读取的，因为，第二次遍历是创建新的Iterator，所以pos也回到了数组起始位置。对于某些InputStream当然可以也这么做。例如：ByteArrayInputStream ，ByteArrayInputStream就是将一个Java的byte数组保存到对象里，然后读取的时候遍历该byte数组。

ByteArrayInputStream的构造函数和字节读取源码：

```java
public ByteArrayInputStream(byte buf[]) {  
        this.buf = buf;  
        this.pos = 0;  
        //假设长度是10，能取的索引是[0,9]
        this.count = buf.length;  
}  

//读取下一个的字节
public synchronized int read() {  
    //pos++，先使用再自增
    return (pos < count) ? (buf[pos++] & 0xff) : -1;  
}  
```

从ByteArrayInputStream源码来看，要实现其重复读取是很简单的，用一个方法重置reset即可，但是为什么没有这个方法，我想是为了遵循InputStream的统一标准，也就是达到末尾后，必须是-1，也就是pos超出索引最大值后返回-1。


InputStream这个抽象类的read方法定义：

```java
/** 
* Reads the next byte of data from the input stream. The value byte is 
* returned as an <code>int</code> in the range <code>0</code> to 
* <code>255</code>. If no byte is available because the end of the stream 
* has been reached, the value <code>-1</code> is returned. This method 
* blocks until input data is available, the end of the stream is detected, 
* or an exception is thrown. 
* 
* <p> A subclass must provide an implementation of this method. 
* 
* @return     the next byte of data, or <code>-1</code> if the end of the 
*             stream is reached. 
* @exception  IOException  if an I/O error occurs. 
*/  
public abstract int read() throws IOException;  
```

InputStream顾名思义就是一个单向的字节流，跟水流一样，要想再次使用就自己再去源头取一下。

InputStream其实不像杯子，更像是一根水管，要想喝水了，就在把水管架在水源与杯子之间，让水流到杯子里（注意：这个动作完成了之后水管里面就没有水了）。

这样看来，**InputStream其实是一个数据通道，只负责数据的流通，并不负责数据的处理和存储等其他工作范畴。**
但是，其实有的InputStream实现类是可以实现数据的处理工作的。但是没有这么做，这就是规范和标准的重要性。

# 3 重复读取InputStream的方法
但是，有些时候，我们必须对字节流进行重复读取，比如：

1. 一个office word文件流，我需要首先读取InputStream中的前一些字节来判断word文件的实际内容（word文件可以保存html，mht的内容）。然后再根据实际内容决定我要解析InputStream的方式。

2. 一个Html文件流，我需要首先读取InputStream中的一些字节来判断Html文件编码方式。然后再根据html文件编码方式读取Html内容

3. 从socket收到的一个InputStream，我首先需要读取InputStream判断是什么类型的字符串。然后再将InputStream读取写到文件里。

总之，在实际的工作当中，我们常常会需要多次读取一个InputStream的需求。
如果该InputStream是我们通过某种方式“主动”获取的，那我们**可以**不必重复读取一个InputStream，而是再次获取一样数据的InputStream来处理。例如： 

```java
InputStream inputStream = new FileInputStream(path);  

//利用inputStream，read操作

//重新获取
inputStream = new FileInputStream(path);  

//再次利用inputStream，read操作
```

再例如：

```java
InputStream inputStream = httpconn.getInputStream();   

//利用inputStream，read操作

//重新获取
inputStream = httpconn.getInputStream();  

//再次利用inputStream，read操作
```

但是，对于“被动”利用InputStream作为入参的方法，在方法内部需要重复利用InputStream，对于InputStream的来源，可能是文件，可能是网络，也可能是内存里的一个String，对于InputStream的具体实现类，方法内部无法知道，因此更谈不上在方法内部重复获取了。比如有这样一个接口： 

```java
//将InputStream转换成一个文本字符串  
public String convert(InputStream inputStream);  
```

在接口的实现中，我们可能需要首先读取InputStream前n个字节来判断InputStream流的数据流型，然后转化InputStream为一个字符串。

## 3.1 缓存法

为了解决上面的问题，最简单的方式就是缓存，首先将InputStream缓存到内存，然后重复使用内存里的数据。

例如专门写一个类用于缓存InputStream到内存中： 

```java
package com.gs.cvoud.attachment.converter;  

import java.io.ByteArrayInputStream;  
import java.io.ByteArrayOutputStream;  
import java.io.IOException;  
import java.io.InputStream;  

import com.gs.cvoud.util.ObjectUtils;  

/** 
 * 缓存InputStream，以便InputStream的重复利用 
 * @author boyce 
 * @version 2014-2-24 
 */  
public class InputStreamCacher {  

    private static final org.slf4j.Logger logger = org.slf4j.LoggerFactory.getLogger(InputStreamCacher.class);  

    /** 
     * 将InputStream中的字节保存到ByteArrayOutputStream中。 
     */  
    private ByteArrayOutputStream byteArrayOutputStream = null;  

    public InputStreamCacher(InputStream inputStream) {  
        if (ObjectUtils.isNull(inputStream))  
            return;  

        byteArrayOutputStream = new ByteArrayOutputStream();  
        byte[] buffer = new byte[1024];    
        int len;    
        try {  
            while ((len = inputStream.read(buffer)) > -1 ) {    
                byteArrayOutputStream.write(buffer, 0, len);    
            }

            //实际上ByteArrayOutputStream自身并重写父类的flush方法 
            //也就是，此处flush实际上调的的是方法
            //也好理解，因为 byteArrayOutputStream 并没destination
            //大概byteArrayOutputStream只是一个缓存用的流,没必要实现flush方法
            byteArrayOutputStream.flush();  

        } catch (IOException e) {  
            logger.error(e.getMessage(), e);  
        }    

    }  

    public InputStream getInputStream() {  
        if (ObjectUtils.isNull(byteArrayOutputStream))  
            return null;  

        return new ByteArrayInputStream(byteArrayOutputStream.toByteArray());  
    }  
}  
```


当在一个方法内部需要重复读取InputpStream时，可以按照如下方式使用上面的InputStreamCacher类：

```java
InputStreamCacher  cacher = new InputStreamCacher(inputStream); 
InputStream stream = cacher.getInputStream();  
//对stream做一些处理

//重新获取stream，准备做一些处理
stream = cacher.getInputStream(); 

```

缓存法是将InputStream缓存到一个ByteArrayOutputStream中，当然缓存的数据类型和方式都是任意的，ByteArrayOutputStream只是一种解决思路。

**这种方式有一个最大的缺点，就是内存压力。**外部传给接口的InputStream有可能很大。每调用一次接口就将InputStream缓存到内存中，内存要承受的压力是可想而知的。

编程永远都是在时间和空间之间找到一个平衡点，前面说的“主动获取方式”的重复获取也有它的缺点，就是需要重新读取文件，获取重新建立网络连接等，这就是需要消耗更多的时间。

## 3.2 mark和reset方法重复利用InputStream
上面的缓存法存在内存消耗过大的缺陷，通过阅读InputStream这个抽象类,发现抽象类有mark和reset方法，看一下的接口的源码：

第一个，InputStream的当前实现类是否支持mark，默认是不支持。

```java
public boolean markSupported() {  
   return false;  
} 
```

第二个，mark接口。该接口在InputStream中默认实现不做任何事情。

```java
public synchronized void mark(int readlimit) {} 
```

第三个，reset接口。该接口在InputStream中实现，直接调用就会抛异常。

```java
public synchronized void reset() throws IOException {  
   throw new IOException("mark/reset not supported");  
}
```

从三个接口定义中可以看出，首先InputStream默认是不支持mark的，子类需要支持mark必须重写这三个方法。

第一个接口很简单，就是标明该InputStream是否支持mark。

### 3.2.1 mark(int readlimit)方法
mark接口的官方文档解释：

在此输入流中标记当前的位置。当调用reset方法后，流开始读取的位置重置为标记的位置，以便后续重新读取相同的字节。

readlimit 参数告知此输入流在标记位置失效之前允许重复读取的字节数。

mark 的常规协定是：

如果方法 markSupported 返回 true，则输入流总会在调用 mark 之后记住所有读取的字节，并且无论何时调用方法 reset ，都会准备再次提供那些相同的字节。但是，如果在调用 reset 之前可以从流中读取多于 readlimit 的字节，则根本不需要该流记住任何数据。

个人理解对readlimit的理解：

假设对InputStream的实例调用了mark(100)，则表明：在调用reset之前


### 3.2.2 reset方法
reset接口的官方文档解释：

将此流重新定位到对此输入流最后调用 mark 方法时的位置。

reset 的常规协定是：

如果方法 markSupported 返回 true，则：

如果创建流以来未调用方法 mark，或最后调用 mark 以来从该流读取的字节数大于最后调用 mark 时的参数，则可能抛出 IOException。

如果未抛出这样的 IOException，则将该流重新设置为这种状态：最近调用 mark 以来（或如果未调用 mark，则从文件开始以来）读取的所有字节将重新提供给 read 方法的后续调用方，后接可能是调用 reset 时的下一输入数据的所有字节。

如果方法 markSupported 返回 false，则：

对 reset 的调用可能抛出 IOException。

如果未抛出 IOException，则将该流重新设置为一种固定状态，该状态取决于输入流的特定类型和其创建方式的固定状态。提供给 read 方法的后续调用方的字节取决于特定类型的输入流。

简而言之就是：

**调用mark方法会记下当前调用mark方法的时刻，InputStream被读到的位置。
调用reset方法就会回到该位置。**

### 3.2.3 举例

```java
String content = "BoyceZhang!";  
InputStream inputStream = new ByteArrayInputStream(content.getBytes());  

// 判断该输入流是否支持mark操作 
// 在c++中，可以通过向上提升的类型操作实现对父类行为的调用，而在Java中，无论对其进行什么样的类型转换，其类型实际上是不变的，只能通过super调用。
//又因为，InputStream是抽象类，markSupported必定是始终调用实现类的markSupported实现
if (!inputStream.markSupported()) {  
    System.out.println("mark/reset not supported!");  
}  

int ch;    
boolean marked = false;    
while ((ch = inputStream.read()) != -1) {  

    //读取一个字符输出一个字符    
    System.out.print((char)ch);    

    //读到 'e' 且 还未进行标记时，标记一下  
     if (((char)ch == 'e')& !marked) { 
        //先不要理会mark的参数     
        inputStream.mark(content.length());  

        marked = true;    
     }    

     //读到'!'的时候重新回到标记位置开始读  
      if ((char)ch == '!' && marked) {    
          inputStream.reset();    
          marked = false;  
      }    
}  

    //程序最终输出：BoyceZhang!Zhang!  
```

看了这个例子之后对mark和reset接口有了很直观的认识。但是mark接口的参数readlimit究竟是干嘛的呢？

我们知道InputStream是不支持mark的。要想支持mark子类必须重写这三个方法，我想说的是不同的实现子类，mark的参数readlimit作用是不同的。常用的FileInputStream并不支持mark。

接下里看一下不同的InputStream实现类的readlimit参数的含义。

#### 3.2.3.1 BufferedInputStream的readlimit
readlimit表示：InputStream调用mark方法的时刻起，在读取readlimit个字节之前，标记的该位置是有效的。如果读取的字节数大于readlimit，可能标记的位置会失效。 

在BufferedInputStream的read方法源码中有这么一段：

```java
} else if (buffer.length >= marklimit) {  
     markpos = -1; // buffer got too big, invalidate mark
     pos = 0; // drop buffer contents
     } else {  /* grow buffer */  
```

为什么是可能会失效呢？因为BufferedInputStream读取不是一个字节一个字节读取的，是一个字节数组一个字节数组读取的，字节数组的大小就是我们自己定义的字节数组大小，一般会无脑定义为4096。

例如，readlimit=35，第一次比较的时候buffer.length=0（没开始读），所以，第一次比较时：0<35。

然后buffer数组的长度是48。这时的read方法只会简单的挨个返回buffer数组中的字节，不会做这次比较。直到读到buffer数组最后一个字节（第48个）后，才重新再次比较。这时如果我们读到buffer中第47个字节就reset。mark仍然是有效的。虽然47>35。 

#### 3.2.3.2 ByteArrayInputStream的readlimit
对于InputStream的另外一个实现类：ByteArrayInputStream，我们发现readlimit参数根本就没有用，调用mark方法的时候写多少都无所谓。

```java
public void mark(int readAheadLimit) {  
   mark = pos;  
}  

public synchronized void reset() {  
   pos = mark;  
}  
```

因为对于ByteArrayInputStream来说，都是通过字节数组创建的，内部本身就保存了整个字节数组，mark只是标记一下数组下标位置，根本不用担心mark会创建太大的buffer字节数组缓存。

所以由于mark和reset方法配合可以记录并回到我们标记的流的位置重新读流，很大一部分就可以解决我们的某些重复读的需要。

这种方式的优点很明显：不用缓存整个InputStream数据。对于ByteArrayInputStream甚至没有任何的内存开销。

当然这种方式也有缺点：就是需要通读InputStream的读取细节，了解底层代码实现，也相对比较复杂。
