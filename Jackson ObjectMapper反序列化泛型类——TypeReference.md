@[TOC]
# 1 代码实践
不比比，看代码即可：
```java
package default_package.objectMapper序列化泛型类;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) throws IOException {
        List<String> list = new ArrayList<String>() {
            //新建一个匿名类实例
            {
                //匿名类的初始化语句，不带static，
                // 但是和static{  }句块的目的相同，都是在构造函数之前执行这些句块中的语句，
                // 只不过static句块专门在静态阶段执行，不带static在新建实例时执行
                add("1");
                add("2");
                add("3");
                add("4");
            }
        };

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true);//美化的json串
        String listJsonString = objectMapper.writeValueAsString(list);
        System.out.println(listJsonString);  //[ "1", "2", "3", "4" ]

        //================================
        List list2=objectMapper.readValue(listJsonString,List.class);
        System.out.println(list2);  //[1, 2, 3, 4]
        System.out.println(list2.get(0).getClass());  //class java.lang.String

        //==============================
        try {
            List<Integer> list3=objectMapper.readValue(listJsonString,List.class);
            System.out.println(list3);  //[1, 2, 3, 4]
            System.out.println(list3.get(0).getClass());  //java.lang.ClassCastException
        }catch (Exception e){
            e.printStackTrace();
        }

        //=============================
        List<String> list4=objectMapper.readValue(listJsonString,new TypeReference<List<String>>(){});
        //第二个参数是用这个类包装了一下，目测是执行时得到TypeReference<T>中T类型，然后得到Class<T>，也就是T.class
        //第二个参数是这个类的匿名实例，然后用实例的getclass之类的方法护着Class<T>

        System.out.println(list4);  //[1, 2, 3, 4]
        System.out.println(list4.get(0).getClass());  //java.lang.ClassCastException

    }
}

```
# 2 总结
1. 其他泛型都可以通过这种方式明确类
2. 第二个参数是TypeReference<T>的匿名实例
