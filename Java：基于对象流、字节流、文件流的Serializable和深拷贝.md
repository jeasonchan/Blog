# 1 Serializable接口概述
有时候我们有一种需求，保存在内存中的各种对象的状态（也就是实例变量，不是方法），并且想把保存的对象状态再读出来，这样既可以实现对象的传递。
实现这一需求的方式有很多，比如，使用objectMapper序列化为Json字符串/对象，再进行传递或者保存，Java的设计指出也考虑到了这种需求，就设计出了Serializable接口，该接口没有必要实现的方法，默认继承了该接口后会就能自动调用java默认的序列化和放序列化。
# 2 Serializable接口代码实践
定义一个类用于序列化：UserInfo .java
```java
@Getter
@Setter
public class UserInfo implements Serializable{ //实现Serializable接口才能被序列化
    private String userName;
    private String usePass;
    private transient int userAge;//使用transient关键字修饰的变量不会被序列化
    public UserInfo() {
        userAge=20;
    }
    public UserInfo(String userName, String usePass, int userAge) {
        super();
        this.userName = userName;
        this.usePass = usePass;
        this.userAge = userAge;
    }

    @Override 
    public String toString() {
        return "UserInfo [userName=" + userName + ", usePass=" + usePass + ",userAge="+(userAge==0?"NOT SET":userAge)+"]";
    }

	 /**
     * 静态方法，序列化对象到文件
     * @param fileName
     */
    public static void serialize(String fileName){
        try {
			//对象输出流和文件输出流结合使用，实现将对象持久化/序列化到本地
            ObjectOutputStream out=new ObjectOutputStream(new FileOutputStream(fileName));
            out.writeObject("序列化的日期是：");//序列化一个字符串到文件
            out.writeObject(new Date());//序列化一个当前日期对象到文件
            UserInfo userInfo=new UserInfo("郭大侠","961012",21);
            out.writeObject(userInfo);//序列化一个会员对象
            out.close();  //资源释放要放到finally中，这里并不规范   
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

	 /**
     * 静态方法，从文件中反序列化对象
     * @param fileName
     */
    public static void deserialize(String fileName){
        try {
            ObjectInputStream in=new ObjectInputStream(new FileInputStream(fileName));
            
            String str=(String) in.readObject();//刚才的字符串对象
            Date date=(Date) in.readObject();//日期对象
            UserInfo userInfo=(UserInfo) in.readObject();//会员对象
            
            System.out.println(str);
            System.out.println(date);
            System.out.println(userInfo);
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

}
```
主方法，运行实验一下：
```java
class Main{
	public static main(String[] args){
		UserInfo.serialize("test");
		sleep(2*60*1000)
        UserInfo.deserialize("test");//这里userAge取读不到是因为使用了transient修饰，所以得到的是默认值
        
        /**
         * UserInfo的无参构造中给userAge属性赋值蛋反序列化得到的结果还是一样。
         * 得出结论：
         * 当从序列化的文件中读出某个类的实例时，实际上并不会执行这个类的构造函数，   
         * 而是载入了一个该类对象的持久化状态，并将这个状态赋值给该类的另一个对象。  
         */
    }
	}
}
```
原始输出：
```
序列化的日期是：
Sun Dec 15 11:13:05 CST 2019
UserInfo [userName=郭大侠, usePass=961012,userAge=NOT SET]
```

# 3 基于字节流、对象流的深拷贝
```java
public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        UserInfo userInfo1 = new UserInfo("123", "456", 111);
        System.out.println(userInfo1);
        UserInfo userInfo2 = deepClone(userInfo1);
        System.out.println(userInfo2);
    }


    @SuppressWarnings("unchecked")
    public static <T> T deepClone(T object) throws IOException, ClassNotFoundException {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(object);

        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
        ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);
        return (T) objectInputStream.readObject();
        
        
        /*
        原始输出：
        UserInfo [userName=123, usePass=456,userAge=111]
        UserInfo [userName=123, usePass=456,userAge=NOT SET]
         */
    }
}
```
从原始输出可以看出，transient 关键字可以可以抑制Object输入输出流对实例被transient 修饰属性的读写。

# 4 总结
以上两种应用场景，本质上Object输入输出流和其他输入输出流的配合使用：
1. 和字节数组输入输入输出流配合使用，实现了对象的深拷贝
2. 和文件输入输出流的配合使用，实现了对象持久化到的本地文件再进行读取的功能
