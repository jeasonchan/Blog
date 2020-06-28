@[TOC]
# JDK配置
windows和linux使用各自可执行文件安装JDk之后，在IDE已经可以使用JDk。
再按需将信息添加到windows的系统环境变量JAVA_PATH和PATH和linux的环境变量中：
* JAVA_HOME这个变量的作用是一些基于java开发的工具会用到，比如tomcat、groovy，如果不用这个工具这个变量也可以免了。具体的值为bin目录的父目录，windows下环境变量值不区分大小写，linux严格区分大小写。 不过通常为了方便以后用java开发的小工具，一般都会设置JAVA_HOME
* %JAVA_HOME%/bin追加到系统环境变量的PATH中，方便命令行识别java和javac命令
# import和package机制
import和include极为不同！！！！

因为 import 的功能仅仅就是导入一个名字，它不像 #include 一样，会将内容载入进来。import 只是请编译器帮你打字，让编译器把没有姓的类别加上姓，并不会把别的文件的程式码写进来。直白一点，你include A之后，A已经include的东西，你就自动拥有了，相当于“继承”了；import A只是相当于编译器会在A路径下去找类。

# 面向接口编程
直接看代码：
先定义一个接口：
```
public interface Calculate {
    int cal(int number1,int number2);
}
```
使用简单工厂，用map实现了接口具体实现类的绑定和调用：
```java
public class CalculateFactory {
	//里面放的是接口，放实现这个接口的类时，相当于会自动转型
    private static Map<String, Calculate> calculateMap = new HashMap<>();  
    public CalculateFactory() {
        calculateMap.put("+", new AddCalculate());
        calculateMap.put("-", new MinusCalculate());
        calculateMap.put("*", new MulCalculate());
        calculateMap.put("/", new DivCalculate());
    }

    public static Calculate getOperator(String operator) {
        if (calculateMap.containsKey(operator)) {
            return calculateMap.get(operator);
        }
        throw new IllegalArgumentException("Wrong input arguement!");
    }


}
```
面向接口编程，使具体的实现方法分散到具体的实现类中，再通过具体实现类的转型为接口从而实现对具体的那个类的调用，以后不需要懂业务代码，只需要更改具体实现类，就可以实现，在使用者“无感知”的情况下更改方法。

# 多线程
同时执行多个任务，叫并发执行任务，并发执行的中的但个任务就是单个线程。图形化一点，从主线程fork线程出去，同时对资源进行操作。

windows中，每一个进程都包含有自身的地址信息，切片式多任务，线程是由进程创建，低一级的单元。

Java中实现多线程的两种方式：thread类和Runnable接口。
用Thread类实现：继承，并重写run()方法
用Runnable接口实现：将Runnable对象作为Thread类的构造参数

线程的生命周期：
1. 出生（线程对象被创建出来的时候，调用start之前）. 
2. 就绪（调用start后，还没得到运行资源时）. sleep和wait会使线程进入此状态
3. 运行（得到资源后，开始执行的状态，可能随时转为后面的四种状态）. 
4. 等待（处于运行状态时，使用wait()方法后进入就绪状态，然后必须用notify()唤醒，notifyall是唤醒所有的线程，重新进入就 绪状态，得到资源后再次进入执行状态）. 
5. 休眠（调用sleep方法后，就被休眠了）. 
6. 阻塞（等待输入输出请求时，进入休眠状态）. 
7. 死亡（run方法执行完毕后，退出run方法的之后，线程死亡）。

线程加入：join()，注意！被join的线程执行完才轮到原先的线程继续执行。

线程中断：线程被interrupt时一定会抛出interrupt异常，但是程序员不作跳出循环的break，线程还是会继续循环执行。

线程礼让：没什么卵用

线程优先级：Thread.MAX_PRIORITY和Thread.MIN_PRIORITY之间，子类线程继承父类线程的优先级，同样的就绪状态的线程，优先级数字越大，越先进入运行状态；并且，高优先级的线程全部执行死亡后，才会分配资源给低优先级的线程。

线程同步：就是解决多线程对同一资源的访问冲突问，使用synchronized关键字修饰代码块或者方法，使被修饰的部分，在同一时间只有一个线程能访问，

多个线程对同一个方法/函数进行调用，如果没用到全局数据并不会有什么影响，因为每一个线程都有自己的线程栈，数据不会串。
	
# 并发编程
杨康，并发编程

jmm，JAVA内存模型

堆是共享的，栈是私有的

单例，七种写法，推荐静态

多线程安全措施主要包含无锁同步和有锁同步，有锁同步即为常见的synchronized和Lock，本质上是一种阻塞，无锁同步（<https://www.cnblogs.com/dengzz/p/5688021.html>）较为高级，主要有三种：
* Atomic，原子类及其原子操作方法
* CAS，
* volatile，详细解析见<https://www.cnblogs.com/aigongsi/archive/2012/04/01/2429166.html>

较为常见的无所同步套路都是由以上三种结合而来，比如：
* While、Volatile、CAS的结合，几乎可以解决任何无锁同步问题，同是，这也是JDK中几乎所有无锁同步的基础，Atomic的操作都是同步此方法进行实现的！

size（）和capacity的比较，如果同时决定加任务，可能导致任务超过capacity，就想test里那样，目前来看，put方法并没有出现这样的问题。
size（）和capacity加入双检查，多冲检测不嫌多

thread. join可以让主线程等fork出的支线程执行玩，而不同sleep

# 文件操作
```java
file.delete();//删除此file对象映射的文件夹或者文件，只能删除空文件夹！！！！有内容的文件夹返回布尔型表示删除失败。
file.mkdir();//使映射对象成为文件夹；无法将文件转为文件夹
file.create();//使映射对象成为文件
file.renameTo(new file(fullPathString));//使用此方法时，必须有一样的父路径
file.getParentFile(); //抽象）父目录的路径映射的file类实例
file.getParent(); //getParentFile().mkdirs(),（抽象）父目录的路径,字符串返回值

```
## 创建目录、文件
```java
private static void createDir(String fullPath) {
    File tempPath = new File(fullPath); // 创建File对象，只是将实际文件夹或者文件的映射为可操作对象
    if (tempPath.exists()) {
        tempPath.delete();//最好对执行结果进行判断！
    }
    tempPath.mkdir(); //通过mkdir()使File实例真正成为目录
}
```
## 重命名目录、文件
```java
public static void changeDirName(String originDirName, String destDirName) {
    File tempPath = new File(originDirName);
    if (tempPath.exists()) {  //映射为File实例
        tempPath.renameTo(new File(destDirName));
        temppath.mkdir();//成为文件夹对象
    } else {
        createDir(destDirName);
    }
}
```
## 实际应用
```java

```
---

## 疑问
是否可以将文件类型的File实例通过mkdir伪装成文件夹？



# 实际应用
## 解压和压缩功能
解压tar包，基于apache的commons-compress实现。

基本思路：从File文件得到一个TarArchiveInputStream（tar文件处理流），遍历处理流中的每个一个Entry，对其进行文件夹、文件判断，然后根据类型新建占位文件，然后从tar处理流中读取数据写入占位文件中，循环往复，直到处理完tar文件处理流中的每一个Entry。
```java
/*
输入：tar包File实例
输出：和tar同级的解压文件，并且返回tar中的文件列表
*/
public static List<String> unTarFile2(File file) throws IOException {
        List<String> fileNamesList = new ArrayList<>();
        String basePath = file.getParent() + File.separator;

        // 新建压输入流，从将压缩文件转化为流待处理
        TarArchiveInputStream tarArchiveInputStream = new TarArchiveInputStream(new FileInputStream(file));
        TarArchiveEntry tarArchiveEntry = null;
        while ((tarArchiveEntry = tarArchiveInputStream.getNextTarEntry()) != null) {
            fileNamesList.add(tarArchiveEntry.getName());
            if (tarArchiveEntry.isDirectory()) {
                new File(basePath + tarArchiveEntry.getName()).mkdirs();
                continue;
            }
            FileOutputStream fileOutputStream = null;

            File realFile = new File(basePath + tarArchiveEntry.getName());
            if (!realFile.getParentFile().exists()) {
                realFile.getParentFile().mkdirs();
            }
            if (!realFile.exists()) {
                realFile.createNewFile();
            }
            fileOutputStream = new FileOutputStream(realFile);
            byte[] bufferArray = new byte[2048];
            int len = -1;
            //从当前压缩流读取数据，并写入到缓存数组
            // 当数组写满时，进入循环体，将缓存数组中的数据写入文件流中，写入结束，进行while循环判断
            //当数组不满，且压缩流读取到末尾时，返回-1，再次进入循环体
            while ((len = tarArchiveInputStream.read(bufferArray)) != -1) {
                fileOutputStream.write(bufferArray, 0, len);
            }
            fileOutputStream.flush();
            fileOutputStream.close();

        }
        tarArchiveInputStream.close();
        // file.delete();  //delete origin tar pack
        return fileNamesList;
    }
```
有空打算改进一下几点：
* 压缩包的情况十分复杂，可以进去之后就只有文件夹，可以全是文件，可以文件夹和文件的混合，这个解压方法只考虑了部分情况
* 圈复杂度较高，异常处理不理想

